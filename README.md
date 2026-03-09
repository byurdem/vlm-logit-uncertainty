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

## Evaluated Models

The experiments analyze the behavior of:

- **VILA-M3**
- **LLaVA-Med**
- **PRISM**

under different prompt complexities and temperature values.

## Installation

Clone the repository:

```bash
git clone https://github.com/USERNAME/vlm-logit-uncertainty.git
cd vlm-logit-uncertainty
````

Install dependencies:

```bash
pip install -r requirements.txt
```

## Usage

Run model inference and uncertainty computation:

```bash
python src/inference.py
```

Compute uncertainty metrics:

```bash
python src/compute_metrics.py
```

## Repository Structure

```
src/                core implementation
notebooks/          analysis notebooks
figures/            plots used in the paper
data/               dataset information
```

## Citation

If you use this work, please cite:

```bibtex
@article{yurdem2026vlmuncertainty,
title={Logit-Level Uncertainty Quantification in Vision-Language Models for Histopathology Image Analysis},
author={Yurdem, Betul and Catak, Ferhat Ozgur and Kuzlu, Murat and Gullu, Mehmet Kemal},
year={2026}
}
```

## License

This project is released under the MIT License.
