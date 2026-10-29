# Logit-Level Uncertainty Quantification in Vision-Language Models

[![arXiv](https://img.shields.io/badge/arXiv-2603.03527-red)]([link](https://arxiv.org/abs/2603.03527))

This repository contains the official implementation of the uncertainty quantification framework proposed in our paper:

"Logit-Level Uncertainty Quantification in Vision-Language Models for Histopathology Image Analysis"

📄 Paper: https://arxiv.org/abs/2603.03527

## Overview

Vision–Language Models (VLMs) have shown strong performance in multimodal tasks. However, their reliability in medical applications remains a critical concern.

This repository provides an uncertainty quantification framework that analyzes temperature-controlled output logits from VLMs.

The framework evaluates uncertainty behavior using:

- Cosine Similarity (CS)
- Jensen–Shannon Divergence (JS)
- Kullback–Leibler Divergence (KL)
- Mean Absolute Error (MAE)

## Dataset
- [ARCH dataset](https://warwick.ac.uk/fac/cross_fac/tia/data/arch/) — PubMed subset
- 150 images were selected using t-SNE on each model's embeddings followed by a selection procedure (`02_image_selection.ipynb`).
  The selected image list is in `selected_images.txt`.

## Evaluated Models
- [VILA-M3 (Llama3-VILA-M3-8B)](https://huggingface.co/MONAI/Llama3-VILA-M3-8B)
- [LLaVA-Med v1.5](https://huggingface.co/microsoft/llava-med-v1.5-mistral-7b)
- [PRISM](https://huggingface.co/paige-ai/Prism) — gated model, access must be requested on Hugging Face

Model weights are not included. Setup instructions are on each model's Hugging Face page.

## Pipeline
1. `01_tsne_*.ipynb` – t-SNE on each model's image embeddings
2. `02_image_selection.ipynb` – selection of 150 images
3. `03_inference_*.ipynb` – running each model on the selected images
4. `04_metrics.ipynb` – evaluation metrics

## Requirements
See `requirements.txt`. A GPU is required.

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
