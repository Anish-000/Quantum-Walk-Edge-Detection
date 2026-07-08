# Diagonal Gradient Extension and Otsu-Based Evaluation for Lackadaisical Quantum Walk Edge Detection

> MCA Thesis Project — University of Kalyani, 2026  
> Extends the quantum walk edge detection method of **Giri et al. (2025)** with two targeted contributions.

---

## Overview

This repository contains the full implementation of my MCA thesis, which extends the lackadaisical discrete-time quantum walk search (DTQWS) edge detection method proposed by Giri, Sato, and Saito (arXiv:2510.04420, 2025).

The paper treats edge detection as a quantum spatial search problem, achieving success probabilities as high as **p_s ≈ 0.98** — far outperforming prior quantum methods like QSobel (0.0095) and HED (0.0031). However, it leaves two important gaps unaddressed:

1. The gradient oracle only checks **4 axis-aligned directions** — missing diagonal edges
2. The post-processing threshold for binarizing the probability map is **never specified**

This thesis addresses both gaps through two verified contributions.

---

## Contributions

### Contribution 1 — Six-Direction Diagonal Gradient Extension

The original gradient oracle computes the maximum intensity difference in 4 directions (right, left, up, down). We extend it to **6 directions** by adding two downward diagonal neighbours:

```
G_max(x,y) = max( |I_h+|, |I_h-|, |I_v+|, |I_v-|, |I(x,y) - I(x+1,y+1)|, |I(x,y) - I(x+1,y-1)| )
```

Since the 6-direction marked set is always a superset of the 4-direction set, recall can only improve. An ablation study confirms that **downward diagonals outperform** upward diagonals and full 8-direction on all 3 metrics.

### Contribution 2 — Otsu-Based Probability Map Thresholding

The paper mentions a threshold for binarizing the probability map but never specifies how to choose it. We replace this with **Otsu's histogram-based method**, which finds the optimal threshold automatically from the probability map's own distribution — requiring **no ground-truth information** at inference time.

We define two evaluation protocols:
- **Otsu Oracle F1** — Otsu applied to the gradient map *before* the quantum walk
- **Otsu Quantum F1** — Otsu applied to the probability map *after* the quantum walk *(primary metric)*

---

## Results

Evaluated on **20 images from the BSDS500 benchmark** using Precision, Recall, and F1 score against human-annotated ground truth.

| Metric | Paper (4-dir) | Proposed (6-dir) | Δ | Improvement |
|---|---|---|---|---|
| Raw Oracle F1 | 0.2571 | **0.2598** | +0.0027 | +1.05% |
| Otsu Oracle F1 | 0.2554 | **0.2664** | +0.0110 | +4.31% |
| Otsu Quantum F1 | 0.2572 | **0.2602** | +0.0030 | +1.17% |

### Showcase Results

| Image | Description | Otsu Oracle Δ | Otsu Quantum Δ |
|---|---|---|---|
| 29030 | Ferrari car (strong diagonal panel edges) | +0.0140 | +0.0080 |
| 8068 | Swan (curved diagonal body contour) | +0.0239 | +0.0071 |

### Ablation Study

| Configuration | Raw Oracle F1 | Otsu Oracle F1 | Otsu Quantum F1 |
|---|---|---|---|
| Paper baseline (4-dir) | 0.2571 | 0.2554 | 0.2572 |
| Upward diagonals | 0.2480 ↓ | 0.2510 ↓ | 0.2484 ↓ |
| Full 8-direction | 0.2525 ↓ | 0.2632 ↑ | 0.2537 ↓ |
| **Downward diagonals (Ours)** | **0.2598 ↑** | **0.2664 ↑** | **0.2602 ↑** |

### Qiskit AerSimulator Results

| Method | t=1 | t=2 | t=3 | t=4 | t=5 |
|---|---|---|---|---|---|
| Paper (4-dir) | 0.1451 | **0.1614** | 0.1350 | 0.0480 | 0.0496 |
| Proposed (6-dir) | 0.1041 | 0.1569 | 0.1315 | **0.0662** | 0.0494 |

> The block circuit peaks at t=2. The diagonal advantage is visible in the full numerical simulation but not in the limited block circuit due to block isolation and insufficient iteration depth.

---

## Repository Structure

```
├── cell_2_paper_full.py          # Paper's baseline DTQWS (4-direction oracle)
│                                 # Parts: DTQWS walk, Raw Oracle F1,
│                                 #        Otsu Oracle F1, Otsu Quantum F1
│
├── cell_3_diagonal_full.py       # Proposed method (6-direction diagonal oracle)
│                                 # Parts: DTQWS walk, Raw Oracle F1,
│                                 #        Otsu Oracle F1, Otsu Quantum F1,
│                                 #        Head-to-head comparison vs paper
│
├── cell_3_diagonal_8dir.py       # Ablation: full 8-direction variant
├── cell_3_diagonal_upward.py     # Ablation: upward diagonal variant
│
├── cell_2_qiskit_block_circuit.py  # Qiskit sanity check (circuit vs numpy)
├── cell_3_qasm_full_image.py       # Full image Qiskit evaluation (paper)
├── cell_3_qasm_diagonal.py         # Full image Qiskit evaluation (diagonal)
│
├── save_thesis_images.py         # Save and download 16 thesis images
│                                 # (8 panels × 2 showcase images)
│
└── README.md
```

---

## Setup and Installation

### Prerequisites

- Python 3.8+
- Google Colab (recommended) or local Jupyter environment

### Install Dependencies

```bash
pip install numpy matplotlib scikit-image scipy qiskit qiskit-aer
```

### Dataset

Download the **BSDS500** dataset from the Berkeley Computer Vision Group:

```
https://bsds.eecs.berkeley.edu
```

Place the images and ground truth `.mat` files in your working directory. The code expects:
- Grayscale images loaded via `imageio` or `PIL`
- Ground truth loaded via `scipy.io.loadmat`

---

## How to Run

All cells are designed to run sequentially in Google Colab or Jupyter Notebook.

### Step 1 — Load Dataset (Cell 1)
Load all 20 BSDS500 images and ground truth maps into `all_images` and `all_ground_truths` dictionaries.

### Step 2 — Run Paper Baseline (Cell 2)
```bash
python cell_2_paper_full.py
```
Outputs: `paper_raw_oracle`, `paper_otsu_oracle`, `paper_otsu_quantum` dictionaries with per-image F1 scores.

### Step 3 — Run Proposed Method (Cell 3)
```bash
python cell_3_diagonal_full.py
```
Outputs: `diagonal_results`, `diagonal_raw_oracle`, `diagonal_otsu_oracle`, `diagonal_otsu_quantum` dictionaries.
Also prints Part 5 — head-to-head comparison table.

### Step 4 — Run Ablation Variants (Optional)
```bash
python cell_3_diagonal_upward.py    # Upward diagonal variant
python cell_3_diagonal_8dir.py      # Full 8-direction variant
```

### Step 5 — Qiskit Circuit Evaluation
```bash
python cell_3_qasm_diagonal.py
```
Builds 16-pattern lookup table on AerSimulator (~15 sec), then evaluates full image.

### Step 6 — Save Thesis Images
```bash
python save_thesis_images.py
```
Saves and downloads 16 image panels (8 per showcase image) for the thesis document.

---

## Experimental Parameters

| Parameter | Symbol | Value |
|---|---|---|
| Self-loop weight | s | 0.0001 |
| Gradient threshold | a_th | 30 |
| Maximum iterations | T | 1000 |
| Dataset | — | BSDS500 |
| Test images | — | 20 |
| Ground truth | — | Consensus of all annotators |
| Otsu implementation | — | `skimage.filters.threshold_otsu` |
| Qiskit shots | — | 20,000 per pattern |

---

## Tech Stack

| Tool | Purpose |
|---|---|
| **Python** | Core implementation language |
| **NumPy** | Full 2D quantum walk statevector simulation |
| **Qiskit** | 4-qubit block circuit implementation |
| **Qiskit Aer** | AerSimulator for circuit execution |
| **scikit-image** | Otsu thresholding (`threshold_otsu`) |
| **SciPy** | Loading BSDS500 ground truth `.mat` files |
| **Matplotlib** | Visualization of gradient maps, probability maps, convergence plots |
| **LaTeX (Overleaf)** | Thesis document |

---

## Method Overview

```
Input Image I(x,y)
        │
        ▼
6-Direction Gradient Oracle (Eq. 3*)     ← Contribution 1
        │
        ▼
Marked Set T_M  (pixels where G_max ≥ 30)
        │
        ▼
DTQWS — Apply U = S · C_G for t iterations
        │
        ▼
Probability Map P(x,y)
        │
        ▼
Otsu Thresholding                        ← Contribution 2
        │
        ▼
Binary Edge Map E(x,y)
```

---

## Citation

If you use this work, please cite the original paper this thesis extends:

```bibtex
@article{giri2025,
  author  = {Giri, Pulak Ranjan and Sato, Rei and Saito, Kazuhiro},
  title   = {Quantum Walk Search Based Edge Detection of Images},
  journal = {arXiv preprint arXiv:2510.04420},
  year    = {2025}
}
```

---

## Author

**Anish Chattopadhyay**  
MCA, Department of Computer Science and Engineering  
University of Kalyani, Kalyani, India  
Roll No. 90/MCA No. 240003  

*Supervised by Mr. Shambo Chatterjee*  
*Department of Computer Science and Engineering, University of Kalyani*

---

## Acknowledgements

- **Giri, Sato, Saito (2025)** for the original DTQWS edge detection framework
- **Berkeley Computer Vision Group** for the BSDS500 dataset
- **Qiskit / IBM** for the open-source quantum computing framework
- **Mr. Shambo Chatterjee** for supervision and guidance throughout this research
