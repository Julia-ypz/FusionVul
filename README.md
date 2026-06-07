# FusionVul: A Multimodal Feature Fusion Framework for Source Code Vulnerability Detection

[![Paper](https://img.shields.io/badge/Paper-JSS%20Accepted-blue)](https://www.journals.elsevier.com/journal-of-systems-and-software)
[![Preprint](https://img.shields.io/badge/Preprint-arXiv-red)](https://arxiv.org/abs/XXXX.XXXXX)

> **Accepted** at the Journal of Systems and Software (Elsevier), 2025.
> Preprint available at: [arXiv:XXXX.XXXXX](https://arxiv.org/abs/XXXX.XXXXX)

> ⚠️ **Placeholder link**: Replace `arXiv:XXXX.XXXXX` with your arXiv submission ID after uploading the preprint.

FusionVul is a multimodal vulnerability detection framework that integrates sequential syntactic representations (via a pretrained Transformer encoder, UniXcoder) with structural semantics propagated through a Gated Graph Neural Network (GGNN) over Code Property Graphs (CPG). A cross-attention-based feature fusion network (CAFFNet) enables fine-grained cross-modal interaction, and a Sample-Aware Weighting (SaW) strategy aggregates multi-branch predictions at inference time.

---

## Table of Contents

- [Overview](#overview)
- [Framework Architecture](#framework-architecture)
- [Code Availability](#code-availability)
- [Requirements](#requirements)
- [Datasets](#datasets)
- [Experimental Setup](#experimental-setup)
- [Results](#results)
- [Citation](#citation)

---

## Overview

Source code vulnerability detection is challenged by the scale, structural complexity, and semantic diversity of modern codebases. FusionVul addresses three core limitations of existing approaches:

1. **Implicit or shallow fusion** — existing multimodal methods fuse features via simple concatenation or at the node-embedding stage, without explicitly modeling cross-modal semantic correspondences.
2. **Fixed branch weighting** — most dual-branch methods apply fixed aggregation weights at inference time, limiting the effective use of fused representations.
3. **Single-modality bias** — sequential (LCS) and structural (CPG) features each capture complementary aspects of program semantics; neither alone is sufficient for diverse vulnerability patterns.

FusionVul consists of three modules:

- **Phase I — Modality-Specific Feature Extraction**: UniXcoder encodes the Linear Code Sequence (LCS); GGNN encodes the Code Property Graph (CPG).
- **Phase II — Cross-Attention Feature Fusion (CAFFNet)**: LCS tokens attend to CPG structural representations via scaled dot-product cross-attention, producing a fused feature $F_{fus}$.
- **Phase III — Sample-Aware Weighted Aggregation (SaW)**: Final prediction is a weighted combination of three branches ($F_{lcs}$, $F_{cpg}$, $F_{fus}$) controlled by coefficients $\mu_1$ and $\mu_2$.

---

## Framework Architecture

```
Source Code
    ├── Linear Code Sequence (LCS)
    │       └── UniXcoder → F_lcs (CLS vector)
    └── Code Property Graph (CPG)  [via Joern]
            └── GGNN + Attention Readout → F_cpg (graph vector)

CAFFNet (Cross-Attention Feature Fusion)
    Q = W_Q · F'_lcs
    K = W_K · F'_cpg
    V = W_V · F'_cpg
    F_fus = ReLU(W_0 · [softmax(QK^T/√d_f)·V ‖ Q] + b_0)

SaW (Sample-Aware Weighted Aggregation)
    Pred = μ1·Pred_lcs + μ2·Pred_cpg + (1-μ1-μ2)·Pred_fus
```

---

## Code Availability

The source code is not publicly available at this time as the research is ongoing. Interested researchers may contact the corresponding author ([ypzhu111@163.com](mailto:ypzhu111@163.com)) at reasonable request.

---

## Requirements

### Hardware

| Component | Specification |
|-----------|--------------|
| GPU | NVIDIA RTX 4090 (24 GB VRAM) |
| System Memory | ≥ 90 GB RAM |

### Software Environment

```
NVIDIA Driver : 550.78
CUDA          : 12.4
Python        : 3.10.12
PyTorch       : 2.4.1
Joern         : v1.1.1700
```

Install Python dependencies:

```bash
pip install torch==2.4.1 --index-url https://download.pytorch.org/whl/cu124
pip install transformers dgl numpy scikit-learn
```

Install Joern for CPG generation:

```bash
# Download Joern v1.1.1700
wget https://github.com/joernio/joern/releases/download/v1.1.1700/joern-cli.zip
unzip joern-cli.zip
```

---

## Datasets

All datasets are function-level vulnerability benchmarks constructed from real-world source code. We use four publicly available datasets:

| Dataset | Projects | CWEs | Total | Vulnerable | Non-Vulnerable | Vul Ratio |
|---------|----------|------|-------|------------|----------------|-----------|
| Devign | 2 | N/A | 26,037 | 11,888 | 14,149 | 45.66% |
| ReVeal | 2 | N/A | 18,169 | 1,664 | 16,505 | 9.16% |
| SVulD | 348 | 91 | 29,256 | 5,260 | 23,996 | 17.97% |
| DiverseVul | 797 | 150 | 349,437 | 18,945 | 330,492 | 5.42% |

> **Note**: SVulD and DiverseVul cover substantially broader source diversity (348 and 797 projects respectively) and explicit CWE annotations (91 and 150 categories), making them more challenging benchmarks for evaluating generalization across diverse vulnerability patterns.

### Download Links

| Dataset | Source | Link |
|---------|--------|------|
| Devign | Google Drive | [Download](https://drive.google.com/file/d/1x6hoF7G-tSYxg8AFybggypLZgMGDNHfF) |
| ReVeal | Google Drive | [Download](https://drive.google.com/drive/folders/1KuIYgFcvWUXheDhT--cBALsfy1I4utOy) |
| SVulD | GitHub | [Download](https://github.com/jacknichao/SVulD/tree/master/Database/SVulD) |
| DiverseVul | Google Drive | [Download](https://drive.google.com/file/d/12IWKhmLhq7qn5B_iXgn5YerOQtkH-6RG/view?usp=sharing) |

Place downloaded datasets under `data/` as follows:

```
data/
├── devign/
├── reveal/
├── svuld/
└── diversevul/
```

### Dataset Licenses and Attribution

These datasets are the original work of their respective authors. We do not redistribute them. If you use any of these datasets, please cite the original papers:

| Dataset | Original Paper | Venue |
|---------|---------------|-------|
| Devign | Zhou et al. (2019) | NeurIPS 2019 |
| ReVeal | Chakraborty et al. (2021) | IEEE Transactions on Software Engineering |
| SVulD | Ni et al. (2023) | ESEC/FSE 2023 |
| DiverseVul | Chen et al. (2023) | RAID 2023 |

> **Note**: All datasets are intended for academic research use only. Please refer to each dataset's original repository for specific terms of use.

```bibtex
@inproceedings{zhou2019devign,
  title     = {Devign: Effective Vulnerability Identification by Learning
               Comprehensive Program Semantics via Graph Neural Networks},
  author    = {Zhou, Yaqin and Liu, Shangqing and Siow, Jingkai and
               Du, Xiaoning and Liu, Yang},
  booktitle = {Advances in Neural Information Processing Systems},
  volume    = {32},
  year      = {2019}
}

@article{chakraborty2021reveal,
  title   = {Deep Learning Based Vulnerability Detection: Are We There Yet?},
  author  = {Chakraborty, Saikat and Krishna, Rahul and Ding, Yangruibo
             and Ray, Baishakhi},
  journal = {IEEE Transactions on Software Engineering},
  volume  = {48},
  pages   = {3280--3296},
  year    = {2021}
}

@inproceedings{ni2023svuld,
  title     = {Distinguishing Look-Alike Innocent and Vulnerable Code by
               Subtle Semantic Representation Learning and Explanation},
  author    = {Ni, Chao and Yin, Xin and Yang, Kailong and Zhao, Dehai and
               Xing, Zhenchang and Xia, Xin},
  booktitle = {Proceedings of the 31st ACM Joint European Software Engineering
               Conference and Symposium on the Foundations of Software Engineering},
  pages     = {1611--1622},
  year      = {2023}
}

@inproceedings{chen2023diversevul,
  title     = {DiverseVul: A New Vulnerable Source Code Dataset for
               Deep Learning Based Vulnerability Detection},
  author    = {Chen, Yizheng and Ding, Zhoujie and Alowain, Lamya and
               Chen, Xinyun and Wagner, David},
  booktitle = {Proceedings of the 26th International Symposium on Research
               in Attacks, Intrusions and Defenses},
  pages     = {654--668},
  year      = {2023}
}
```

---

## Experimental Setup

### Data Split

All datasets are randomly split into training, validation, and test sets at a **6:2:2** ratio. No class-balancing (e.g., resampling) is applied; the original class distributions are preserved to maintain comparability with prior work.

### CPG Generation

CPGs are generated using **Joern v1.1.1700**. Joern imposes no fixed upper bound on input code length and processes each function without truncation.

```bash
joern-parse <source_dir> --output <cpg_output_dir>
```

### Hyperparameters

| Parameter | Value |
|-----------|-------|
| Epochs | 100 |
| Batch size | 32 |
| Optimizer | Adam |
| Learning rate | 1e-4 |
| Loss function | Cross-entropy |
| UniXcoder max token length | 512 (truncated to first 512 tokens) |
| UniXcoder weights | Frozen during training |
| SaW coefficient search space | {0, 0.1, 0.2, ..., 0.9, 1.0} |
| SaW constraint | μ1 + μ2 ≤ 1 |

The SaW coefficients $\mu_1$ (LCS branch weight) and $\mu_2$ (CPG branch weight) are selected via **grid search** over the discrete set above. The fused branch receives the remaining weight $1 - \mu_1 - \mu_2$. Optimal configurations are dataset-dependent:

- **Devign**: $\mu_1 = 0.3$, $\mu_2 = 0.1$ (balanced, moderate weights across branches)
- **SVulD**: $\mu_1 = 0.1$, $\mu_2 = 0.5$ (higher structural branch weight, reflecting CPG-dominant separability)

---

## Results

### Comparison with Baselines (F1 Score)

| Method | Devign | ReVeal | SVulD | DiverseVul |
|--------|--------|--------|-------|------------|
| LineVul | 54.07 | 49.10 | 25.58 | 11.04 |
| Devign (GGNN) | 54.58 | 33.91 | 16.29 | 9.28 |
| Reveal | 56.59 | 33.87 | 19.31 | 12.42 |
| Vul-LMGNNs | 58.33 | 38.27 | 50.86 | 23.52 |
| SCALE | **65.11** | **62.85** | 32.33 | 11.68 |
| CSGVD | 60.05 | 43.68 | 48.94 | 20.65 |
| **FusionVul** | 58.42 | 47.09 | **53.86** | **25.12** |

FusionVul achieves the **best F1 on SVulD and DiverseVul**, where function size distributions are highly dispersed and vulnerability-type coverage is broad. On Devign and ReVeal, where LCS embeddings show higher separability, SCALE (sequence-only) attains higher recall and F1 due to stronger inductive bias toward token-level patterns.

### Ablation Study

| Method | Devign F1 | ReVeal F1 | SVulD F1 | DiverseVul F1 |
|--------|-----------|-----------|----------|---------------|
| FusionVul w/o GGNN | 57.36 | 43.65 | 51.00 | 23.17 |
| FusionVul w/o UniXcoder | 17.87 | 13.60 | 29.33 | 14.00 |
| **FusionVul (full)** | **58.42** | **47.09** | **53.86** | **25.12** |

---

## Citation

If you find this work useful, please cite:

```bibtex
@article{yang2025fusionvul,
  title     = {FusionVul: A Multimodal Feature Fusion Framework for Source Code Vulnerability Detection},
  author    = {Yang, Hongyu and Zhu, Yaping and Luo, Jingchuan and Nomaguchi, Hiroshi and Su, Chunhua and Susilo, Willy},
  journal   = {Journal of Systems and Software},
  publisher = {Elsevier},
  year      = {2025},
  doi       = {10.1016/j.jss.XXXX.XXXXXX}
}
```

> ⚠️ Replace `10.1016/j.jss.XXXX.XXXXXX` with the actual DOI after formal publication.

---

## Acknowledgments

This work was supported by the Civil Aviation Joint Research Fund Project of the National Natural Science Foundation of China [grant number U2433205].
