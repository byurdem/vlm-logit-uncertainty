# Logit-Level Uncertainty Quantification in Vision-Language Models for Histopathology Image Analysis

<!--
[![arXiv](https://img.shields.io/badge/arXiv-2603.03527-red)]([link](https://arxiv.org/abs/2603.03527))
-->
This repository contains the official implementation of the uncertainty quantification framework proposed in our paper:

"Logit-Level Uncertainty Quantification in Vision-Language Models for Histopathology Image Analysis"

📄 Paper: <!-- https://arxiv.org/abs/2603.03527-->

## Overview

Vision–Language Models (VLMs) have shown strong performance in multimodal tasks. However, their reliability in medical applications remains a critical concern. This repository provides an uncertainty quantification framework that analyzes temperature-controlled output logits from VLMs.

The framework evaluates uncertainty behavior using:

- Cosine Similarity (CS)
- Jensen–Shannon Divergence (JS)
- Kullback–Leibler Divergence (KL)
- Mean Absolute Error (MAE)

## Dataset
- [ARCH dataset](https://warwick.ac.uk/fac/cross_fac/tia/data/arch/) — PubMed subset
- Images were selected using t-SNE on each model's embeddings followed by a selection procedure. The selected image list is in `image_list.txt`.
- The images are not included in this repository. Please download them from the link above, and cite the ARCH dataset as described on its page.

## Evaluated Models
- [VILA-M3-8B](https://huggingface.co/MONAI/Llama3-VILA-M3-8B)
- [LLaVA-Med v1.5](https://huggingface.co/microsoft/llava-med-v1.5-mistral-7b)
- [PRISM](https://huggingface.co/paige-ai/Prism) — gated model, access must be requested on Hugging Face

Model weights are not included. Setup instructions are on each model's Hugging Face page.


## Pipeline
```
           ARCH dataset (PubMed subset, 3,309 images)
                              │
   ┌──────────────────────────┼──────────────────────────┐
   ▼                          ▼                          ▼
VILA-M3                   LLaVA-Med                    PRISM
(vila_m3_uq.ipynb)        (llava_med_uq.ipynb)         (prism_uq.ipynb)
   │  Step 1: image embeddings → 2D t-SNE → *_embeddings_2d.xlsx
   └──────────────────────────┬──────────────────────────┘
                              ▼
        Step 2: image selection (embedding_space.ipynb)
                    → image_list.txt
                              │
   ┌──────────────────────────┼──────────────────────────┐
   ▼                          ▼                          ▼
       Step 3: run each model on the selected images
 3 questions × 11 temperatures × 30 runs → .pkl files (logits)
   └──────────────────────────┬──────────────────────────┘
                              ▼
        Step 4: metrics and plots (calculate_metrics.ipynb)
```
Note that the model notebooks are used twice: first for Step 1 (embeddings), then for Step 3 (model runs), with Step 2 in between.

## Repository contents

| File | Purpose |
|------|---------|
| `vila_m3_uq.ipynb` | VILA-M3: model loading, embedding extraction + t-SNE (Step 1), repeated runs (Step 3) |
| `llava_med_uq.ipynb` | LLaVA-Med: same structure as above |
| `prism_uq.ipynb` | PRISM: Virchow tile embeddings, t-SNE (Step 1), repeated runs (Step 3) |
| `embedding_space.ipynb` | Selecting a diverse image subset and visualizing it in each embedding space (Step 2) |
| `calculate_metrics.ipynb` | Pairwise metrics between runs, aggregation, normalization and plots (Step 4) |
| `image_list.txt` | IDs of the selected images (one ARCH image ID per line) |

## Step-by-step description

### Step 1 – Embedding extraction and t-SNE (model notebooks)

For each model, every image in the dataset is passed through the model's vision encoder only (no text generation), and one feature vector per image is taken:

- VILA-M3-8B and LLaVA-Med v1.5: the first token of the vision encoder output.
- PRISM: Virchow tile embedding = class token concatenated with the mean of the patch tokens. These embeddings are saved as `.pth` files because PRISM uses them as its input.

The vectors are reduced to 2D with t-SNE and saved as `vila_embeddings_2d.xlsx`, `llavamed_embeddings_2d.xlsx`, `prism_embeddings_2d.xlsx`.

### Step 2 – Image selection (`embedding_space.ipynb`)

To test the models on images that cover the whole variety of the dataset, images are selected with Farthest Point Sampling (FPS) in the 2D embedding spaces: at each step the image farthest from all already-selected images is added. Because each model sees the images differently, the selection takes all three embedding spaces into account.

The final list is `image_list.txt`. It was built in several rounds (each round kept the previous images and added new ones). The notebook reproduces this procedure in one place: given an existing list, it selects any number of additional images, saves the combined list and plots the selected images over the density of all images for each model.

### Step 3 – Repeated model runs (model notebooks)

Each model is run on the selected images with three questions (prompts):

| ID | Question type | Content |
|----|---------------|---------|
| Q1 | Basic morphology | cell types, tissue organization, signs of malignancy or benign change |
| Q2 | Diagnostic evaluation | diagnosis, grade, staging features, prognostic indicators |
| Q3 | Quantitative analysis | cell proportions, N/C ratio, mitoses, pleomorphism, architecture, inflammation |

The full prompt texts are in the notebooks (`prompts[1]`, `prompts[2]`, `prompts[3]`). The notebooks also define paraphrased versions of each question (IDs 11–13, 21–23, 31–33). 

For every **image × question × temperature** (temperatures 0.0, 0.1, …, 1.0), the model is run **30 times**, and the logits of each run are saved to a separate `.pkl` file.

- **VILA-M3-8B and LLaVA-Med v1.5** generate the answer token by token (up to 512 tokens). At temperature 0.0 the most likely token is always chosen (greedy). Above 0, the next token is sampled with temperature and top-p = 0.9. The logits of every generation step are saved. A tensor of shape `[1, number_of_generated_tokens, vocabulary_size]`.
- **PRISM** produces logits in a single forward pass from the prompt tokens and the Virchow tile embedding. Run-to-run variation is introduced by perturbing these logits in proportion to the temperature (Gaussian noise plus a boost on a token sampled with top-p = 0.9. The `aggressive` method in the notebook). At temperature 0 the logits are not changed.

Output folder layout:

```
pkl_outputs_{MODEL}_q{question}/
    pkl_output_{image_index}/
        ..._img_{image_id}_temp_{temperature}_{timestamp}.pkl   # {"logits": tensor}
```

`image_index` is the position of the image in `image_list.txt` (starting from 1), and the
image ID is also part of each file name.

### Step 4 – Metrics and plots (`calculate_metrics.ipynb`)

For one image, one question and one temperature there are 30 runs. Every pair of runs is compared (after truncating both to the shorter length):

| Metric | Meaning | Consistent model → |
|--------|---------|--------------------|
| `cosine_flat` | cosine similarity of the flattened logits | high |
| `kl_ab` | KL divergence between the token probability distributions | low |
| `js_divergence` | Jensen–Shannon divergence (symmetric version of KL) | low |
| `mae` | mean absolute difference of the logits | low |

The notebook then:

1. saves one Excel file per image (`metrics_q{q}_img{n}.xlsx`, one sheet per temperature),
2. averages the metrics per image and temperature, then computes the mean and standard deviation over images for each model, question and temperature,
3. min–max normalizes each metric per model (so models with different vocabularies can be compared on the same 0–1 scale),
4. plots each metric against temperature for each model and question.

## Environment

The experiments were run on 2 × NVIDIA RTX A5000 (24 GB) with PyTorch 2.5.1 (CUDA 12.1). The models need separate environments because of different dependencies:

- LLaVA-Med: `transformers==4.41.0` + LLaVA-Med code
- PRISM: `transformers==4.53.0`, `timm`, `xformers` (optional)
- VILA-M3: the environment of VLM-Radiology-Agent-Framework

Analysis notebooks (`embedding_space.ipynb`, `calculate_metrics.ipynb`) need only: `numpy`, `pandas`, `scipy`, `matplotlib`, `seaborn`, `openpyxl`, `xlsxwriter`, `torch` (CPU is enough).

## Citation

If you use this repo, please cite:

```bibtex
@article{yurdem2026vlmuncertainty,
  title={Logit-Level Uncertainty Quantification in Vision-Language Models for Histopathology Image Analysis},
  author={Yurdem, Betul and Catak, Ferhat Ozgur and Kuzlu, Murat and Gullu, Mehmet Kemal},
  journal={arXiv preprint arXiv:2603.03527},
  year={2026}
}
```

## License

This project is released under the MIT License.
