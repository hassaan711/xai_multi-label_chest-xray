# An Empirical Investigation of Explainable AI Methods for Multi-Label Chest X-Ray Diagnosis

[![Python 3.10+](https://img.shields.io/badge/python-3.10+-blue.svg)](https://www.python.org/)
[![PyTorch 2.0+](https://img.shields.io/badge/PyTorch-2.0+-ee4c2c.svg)](https://pytorch.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

This repository contains all code for the empirical study comparing five XAI methods — **LayerCAM**, **FPN-LayerCAM**, **ScoreCAM**, **FPN-ScoreCAM**, and **LIME** — applied to a DenseNet169 classifier trained on NIH ChestX-ray14 and CheXpert, evaluated on the CheXlocalize radiologist-annotated benchmark.

---

## 📋 Contents

```
repo/
├── notebooks/
│   ├── 01_densenet169_chestxray14.ipynb   # Stage 1: training on NIH ChestX-ray14
│   ├── 02_densenet169_chexpert.ipynb       # Stage 2: fine-tuning on CheXpert
│   ├── 03_xai_heatmaps.ipynb              # Generate XAI visual grids (100 samples)
│   ├── 04_xai_evaluation_pilot.ipynb      # Pilot evaluation: 100-image subset
│   ├── 05_xai_evaluation_expanded.ipynb   # Full evaluation: 518 CheXlocalize + 400 NIH
│   ├── 06_fpn_weight_ablation.ipynb       # FPN pyramid weight ablation study
│   └── 07_gt_mask_rebuild.ipynb           # GT mask generation + XAI grid rebuild
├── results/                               # Pre-computed CSVs (populate after running)
├── requirements.txt
└── README.md
```

---

## 🗂️ Datasets

You must download these datasets separately. They are **not included** in this repository.

| Dataset | Source | Notebooks that use it |
|---|---|---|
| **NIH ChestX-ray14** | [NIH Clinical Center](https://nihcc.app.box.com/v/ChestXray-NIHCC) | 01, 03, 04, 05 |
| **CheXpert v1.0** | [Stanford ML Group](https://stanfordmlgroup.github.io/competitions/chexpert/) | 02 |
| **CheXlocalize** | [GitHub](https://github.com/rajpurkarlab/cheXlocalize) | 05, 06, 07 |

### Expected directory layout

**ChestX-ray14:**
```
/path/to/ChestX-ray14/
├── Data_Entry_2017_v2020.csv
├── images_001/images/*.png
├── images_002/images/*.png
│   ...
└── images_012/images/*.png
```

**CheXpert:**
```
/path/to/CheXpert/
└── CheXpert-v1.0/
    ├── train.csv
    ├── valid.csv
    └── train/patient00001/study1/view1_frontal.jpg  ...
```

**CheXlocalize:**
```
/path/to/CheXlocalize/
├── test_labels.csv
├── gt_annotations_test.json        # polygon annotations
└── <images referenced by Path column in test_labels.csv>
```

---

## ⚙️ Installation

```bash
# 1. Clone the repo
git clone https://github.com/<your-username>/xai-chest-xray.git
cd xai-chest-xray

# 2. Create a virtual environment (recommended)
python -m venv venv
source venv/bin/activate        # Linux / macOS
# venv\Scripts\activate         # Windows

# 3. Install dependencies
pip install -r requirements.txt

# 4. (GPU) Install the CUDA-enabled PyTorch build if needed
# See https://pytorch.org/get-started/locally/ for your CUDA version
# Example for CUDA 12.1:
# pip install torch torchvision --index-url https://download.pytorch.org/whl/cu121

# 5. Launch JupyterLab
jupyter lab
```

> **Hardware note:** All notebooks were developed and tested on a single 8 GB VRAM GPU. Mixed precision (AMP) is enabled by default. If you encounter OOM errors, reduce `BATCH_SIZE` from 32 to 16 in the configuration cells.

---

## 🚀 Running the Notebooks

Run them **in order**. Each notebook saves outputs (checkpoints, CSVs, images) that the next notebook expects.

### Notebook 01 — ChestX-ray14 Training

**Input:** `Data_Entry_2017_v2020.csv` + images  
**Output:** `./checkpoints/best_model.pth`, `test_metrics.csv`, learning curve & ROC plots  
**Key settings to change:** `DATASET_ROOT` in cell 2

```python
DATASET_ROOT = Path('/path/to/ChestX-ray14')   # ← change this
```

**Expected result:** Macro AUC ≈ 0.829, early stop at epoch ~16

---

### Notebook 02 — CheXpert Fine-tuning

**Input:** CheXpert CSVs + images + `./checkpoints/best_model.pth` from notebook 01  
**Output:** `./chexpert_output/best_model.pth`, test metrics on CheXlocalize  
**Key settings to change:** `CHEXPERT_ROOT`, `CHEXLOCALIZE_ROOT` in cell 2

```python
CHEXPERT_ROOT     = Path('/path/to/CheXpert')
CHEXLOCALIZE_ROOT = Path('/path/to/CheXlocalize')
PRETRAINED_CKPT   = Path('./checkpoints/best_model.pth')
```

**Expected result:** Competition-5 AUC ≈ 0.856, Val AUC best ≈ 0.862

---

### Notebook 03 — XAI Visual Heatmaps

**Input:** CheXpert fine-tuned checkpoint, 100 sample images  
**Output:** Multi-panel XAI grid images (`./xai_output/`)  
**Key settings:** `CKPT_PATH`, `IMAGE_DIR`, `N_SAMPLES`

---

### Notebook 04 — Pilot XAI Evaluation (100 images)

**Input:** 100-image stratified sample, checkpoint  
**Output:** `./xai_evaluation/results_pilot.csv`, `summary_pilot.csv`  

This notebook validates the evaluation framework before running the full-scale study.

---

### Notebook 05 — Expanded XAI Evaluation (518 + 400 images)

**Input:** Full CheXlocalize frontal set (518), NIH test split (400), checkpoint  
**Output:**
- `./xai_evaluation_full/results_chexlocalize_413.csv`
- `./xai_evaluation_full/results_chestxray14_400.csv`
- `./xai_evaluation_full/summary_chexlocalize_413.csv`
- `./xai_evaluation_full/summary_chestxray14_400.csv`

**Key settings:** `CKPT_PATH`, `CHEXLOCALIZE_DIR`, `NIH_IMAGE_INDEX`

> Resume-friendly: the runner skips images already present in the output CSV. Safe to interrupt and restart.

---

### Notebook 06 — FPN Weight Ablation

**Input:** Results CSV from notebook 05 (used as original-3level reference), checkpoint, images  
**Output:**
- `./xai_ablation/ablation_results.csv`
- `./xai_ablation/ablation_summary_FPNLayerCAM.csv`
- `./xai_ablation/ablation_summary_FPNScoreCAM.csv`
- Bar chart and mIoU heatmap figures

**Three weight configurations tested:**

| Config | db1 | db2 | db3 | db4 |
|---|---|---|---|---|
| Equal | 0.25 | 0.25 | 0.25 | 0.25 |
| Depth-proportional | 0.10 | 0.20 | 0.30 | 0.40 |
| Reverse | 0.40 | 0.30 | 0.20 | 0.10 |

---

### Notebook 07 — GT Mask Generation & Grid Rebuild

**Input:** `gt_annotations_test.json`, original XAI grid images, results CSVs  
**Output:**
- `./gt_masks/*_gt_mask.png` — standalone GT mask overlays
- `./xai_output_with_gt/*_xai_with_gt.png` — rebuilt 6-panel grids with GT column

---

## 📊 Key Results

### Classification

| Stage | Dataset | Macro AUC | Competition-5 AUC |
|---|---|---|---|
| Stage 1 | NIH ChestX-ray14 (test) | 0.829 | — |
| Stage 2 | CheXlocalize (test) | 0.826 | 0.856 |

### XAI Evaluation — CheXlocalize 518 images

| Method | PG ↑ | IoU@0.5 ↑ | mIoU ↑ | Del AUC ↓ | Ins AUC ↑ |
|---|---|---|---|---|---|
| LayerCAM | 0.406 | 0.256 | **0.194** | 0.473 | 0.672 |
| **FPN-LayerCAM** | 0.601 | **0.258** | 0.170 | **0.471** | **0.681** |
| ScoreCAM | 0.527 | 0.092 | 0.080 | 0.498 | 0.632 |
| FPN-ScoreCAM | 0.257 | 0.085 | 0.071 | 0.494 | 0.622 |
| LIME | **0.686** | 0.117 | 0.108 | 0.441 | 0.640 |

### FPN Ablation — Best Config (FPN-LayerCAM, Depth-proportional)

| Metric | Original-3level | Depth-proportional | Δ |
|---|---|---|---|
| Pointing Game | 0.601 | 0.672 | **+11.7%** |
| IoU@0.5 | 0.258 | 0.284 | **+10.1%** |
| mIoU | 0.170 | 0.198 | **+16.3%** |

> **Key finding:** FPN fusion is beneficial for gradient-based LayerCAM (depth-proportional +16.3% mIoU) but catastrophically degrades gradient-free ScoreCAM under reversed pyramid weights (IoU@0.5 −83.7%). This identifies gradient directionality as a necessary structural condition for constructive multi-scale saliency fusion.

---

## 🏗️ Model Architecture

```
DenseNet169 (ImageNet pretrained)
└── features
    ├── denseblock1  →  56×56, 256 ch   (stride ×4)
    ├── denseblock2  →  28×28, 512 ch   (stride ×8)
    ├── denseblock3  →  14×14, 1280 ch  (stride ×16)
    └── denseblock4  →   7×7, 1664 ch   (stride ×32)
└── classifier: Sequential(Dropout(0.5), Linear(1664, 14))
```

FPN-LayerCAM and FPN-ScoreCAM fuse maps from all four dense blocks using configurable pyramid weights.

---

## 🗃️ Checkpoint Paths

| Checkpoint | Notebook that saves it | Notebooks that load it |
|---|---|---|
| `./checkpoints/best_model.pth` | 01 | 02, 03, 04, 05, 06, 07 |
| `./chexpert_output/best_model.pth` | 02 | 03, 04, 05, 06, 07 |

> All XAI notebooks (03–07) use the **CheXpert fine-tuned** checkpoint by default.

---

## 📂 Output Files Reference

```
./checkpoints/
    best_model.pth                        # ChestX-ray14 best checkpoint
./chexpert_output/
    best_model.pth                        # CheXpert fine-tuned checkpoint
    test_metrics.csv
./xai_output/
    chexlocalize/*.png                    # 6-panel XAI grids
    chestxray14/*.png
./xai_evaluation_full/
    results_chexlocalize_413.csv          # Per-image, per-method metrics
    results_chestxray14_400.csv
    summary_chexlocalize_413.csv          # Aggregated summary
    summary_chestxray14_400.csv
./xai_ablation/
    ablation_results.csv
    ablation_summary_FPNLayerCAM.csv
    ablation_summary_FPNScoreCAM.csv
./gt_masks/
    *_gt_mask.png                         # Standalone GT overlay images
./xai_output_with_gt/
    *_xai_with_gt.png                     # Rebuilt grids including GT column
```

---

## 🔬 Reproducibility

All random operations use `SEED = 42`. The patient-level split uses `GroupShuffleSplit(test_size=0.20, random_state=42)`. To reproduce the exact NIH test split used in XAI evaluation, call `reproduce_nih_test_split()` in notebook 05 — it replicates the same split from the CSV without requiring the original split files.

```python
# In notebook 05, cell 3:
SEED = 42
seed_everything(SEED)
```

---

## 📚 Citation

If you use this code in your research, please cite:

```bibtex
@article{author2026xai_chestxray,
  title   = {An Empirical Investigation of Explainable AI Methods for Multi-Label Chest X-Ray Diagnosis},
  author  = {[Authors]},
  journal = {European Journal of Radiology},
  year    = {2026}
}
```

---

## 📜 License

This project is released under the MIT License. See [LICENSE](LICENSE) for details.

The datasets (NIH ChestX-ray14, CheXpert, CheXlocalize) are subject to their own respective licenses and terms of use. Please ensure compliance before downloading or using them.
