# GitHub Repository Setup Guide

Step-by-step instructions to create the public GitHub repository for this project
and push all code files to it.

---

## Prerequisites

- A GitHub account (free at https://github.com)
- Git installed on your machine (`git --version` to check)
- The repo folder from this study already on your local machine

---

## Part 1 — Create the Repository on GitHub

1. Go to **https://github.com/new**

2. Fill in the form:
   - **Repository name:** `xai-chest-xray`  
     *(or any name you prefer, e.g. `chest-xray-xai-study`)*
   - **Description:** `Empirical comparison of XAI methods (LayerCAM, FPN-LayerCAM, ScoreCAM, FPN-ScoreCAM, LIME) for multi-label chest X-ray diagnosis with DenseNet169`
   - **Visibility:** Public *(required if you want to cite it in a journal paper)*
   - ☑ **Add a README file** — **uncheck this** (you already have one)
   - ☑ **Add .gitignore** — select **Python** from the dropdown
   - ☑ **Choose a license** — select **MIT License**

3. Click **Create repository**

4. Copy the repository URL shown on the next page, e.g.:
   ```
   https://github.com/your-username/xai-chest-xray.git
   ```

---

## Part 2 — Set Up Git Locally

Open a terminal in the folder containing this study's code files.

```bash
# Navigate to the repo folder
cd /path/to/repo          # replace with the actual path on your machine

# Initialise git (skip if already a git repo)
git init

# Set your identity (only needed once per machine)
git config --global user.name  "Your Name"
git config --global user.email "your@email.com"

# Connect to the remote GitHub repo you just created
git remote add origin https://github.com/your-username/xai-chest-xray.git

# Verify the remote was added
git remote -v
```

---

## Part 3 — Add a .gitignore

Create a `.gitignore` file to exclude large files and system artefacts:

```bash
cat > .gitignore << 'EOF'
# Python
__pycache__/
*.py[cod]
*.egg
*.egg-info/
dist/
build/
.env
venv/
.venv/

# Jupyter
.ipynb_checkpoints/
*.ipynb_checkpoints

# Data — never commit raw datasets or images
*.png
*.jpg
*.jpeg
*.dcm
*.nii
*.csv
*.pth        # model checkpoints — too large for GitHub
*.pt

# Exceptions: commit the results CSVs but not image outputs
!results/*.csv

# OS
.DS_Store
Thumbs.db

# IDE
.vscode/
.idea/
EOF
```

> **Important:** Never commit dataset images or model checkpoints to GitHub — they are gigabytes in size. Store checkpoints separately (e.g. Google Drive, Hugging Face Hub, university storage) and link to them in the README.

---

## Part 4 — Commit and Push

```bash
# Stage all files
git add .

# Review what will be committed
git status

# First commit
git commit -m "Initial commit: all 7 notebooks, requirements.txt, README"

# Set default branch name to 'main' (GitHub default)
git branch -M main

# Push to GitHub
git push -u origin main
```

If prompted for credentials:
- **Username:** your GitHub username
- **Password:** use a **Personal Access Token** (PAT), not your password.  
  Create one at: https://github.com/settings/tokens → "Generate new token (classic)"  
  Required scope: **repo**

---

## Part 5 — Store Checkpoints Externally (Recommended)

GitHub has a 100 MB file size limit. Model checkpoints are typically 80–120 MB. The recommended approach:

### Option A — Google Drive (simplest)

1. Upload both checkpoint files to Google Drive:
   - `checkpoints/best_model.pth`  (NIH ChestX-ray14)
   - `chexpert_output/best_model.pth`  (CheXpert fine-tuned)

2. Right-click each → "Share" → "Anyone with the link" → "Viewer"

3. Add download links to your `README.md`:
   ```markdown
   ## Pre-trained Checkpoints
   
   Download and place in the corresponding directories:
   | Checkpoint | Description | Download |
   |---|---|---|
   | `checkpoints/best_model.pth` | DenseNet169 — ChestX-ray14 | [Google Drive](YOUR_LINK) |
   | `chexpert_output/best_model.pth` | DenseNet169 — CheXpert fine-tuned | [Google Drive](YOUR_LINK) |
   ```

### Option B — Hugging Face Hub (academic preferred)

```bash
pip install huggingface_hub

python3 - << 'EOF'
from huggingface_hub import HfApi
api = HfApi()
api.upload_file(
    path_or_fileobj="checkpoints/best_model.pth",
    path_in_repo="checkpoints/best_model.pth",
    repo_id="your-hf-username/xai-chest-xray",
    repo_type="model",
)
EOF
```

Then reference in README:
```markdown
```python
from huggingface_hub import hf_hub_download
path = hf_hub_download("your-hf-username/xai-chest-xray", "checkpoints/best_model.pth")
```
```

---

## Part 6 — Add Results CSVs (Optional but Recommended)

Pre-computed evaluation results allow readers to reproduce figures without re-running experiments:

```bash
# Create results directory and add the CSVs
mkdir -p results
cp xai_evaluation_full/summary_chexlocalize_413.csv  results/
cp xai_evaluation_full/summary_chestxray14_400.csv   results/
cp xai_ablation/ablation_summary_FPNLayerCAM.csv     results/
cp xai_ablation/ablation_summary_FPNScoreCAM.csv     results/

# Stage and commit
git add results/
git commit -m "Add pre-computed evaluation results CSVs"
git push
```

---

## Part 7 — Final Repository Checklist

Before sharing the repository URL with your supervisors or submitting to a journal, verify:

- [ ] `README.md` has been updated with the correct GitHub username in clone commands
- [ ] Checkpoint download links (Google Drive or HuggingFace) are working
- [ ] `.gitignore` correctly excludes all `.png`, `.jpg`, `.pth`, `.pt` files
- [ ] All 7 notebooks open and display correctly in GitHub's notebook preview
- [ ] `requirements.txt` lists all dependencies
- [ ] Repository is set to **Public**
- [ ] Dataset attribution and license compliance noted in README

---

## Quick Reference — Common Git Commands

```bash
# Check current status
git status

# See full commit history
git log --oneline

# Add a single file
git add notebooks/05_xai_evaluation_expanded.ipynb

# Amend the last commit message (before pushing)
git commit --amend -m "New message"

# Push changes after additional commits
git push

# Pull latest changes from GitHub (e.g. if you edited README on the website)
git pull

# Create and switch to a new branch
git checkout -b feature/new-analysis

# Merge branch back to main
git checkout main
git merge feature/new-analysis
```

---

## Recommended Repository Structure After Setup

```
xai-chest-xray/                          ← GitHub root
├── .gitignore
├── LICENSE
├── README.md
├── GITHUB_SETUP.md
├── requirements.txt
├── notebooks/
│   ├── 01_densenet169_chestxray14.ipynb
│   ├── 02_densenet169_chexpert.ipynb
│   ├── 03_xai_heatmaps.ipynb
│   ├── 04_xai_evaluation_pilot.ipynb
│   ├── 05_xai_evaluation_expanded.ipynb
│   ├── 06_fpn_weight_ablation.ipynb
│   └── 07_gt_mask_rebuild.ipynb
└── results/
    ├── summary_chexlocalize_413.csv
    ├── summary_chestxray14_400.csv
    ├── ablation_summary_FPNLayerCAM.csv
    └── ablation_summary_FPNScoreCAM.csv
```

Not committed (stored externally or locally only):
```
checkpoints/best_model.pth               ← link in README
chexpert_output/best_model.pth           ← link in README
xai_output/**/*.png                      ← excluded by .gitignore
gt_masks/*.png                           ← excluded by .gitignore
xai_output_with_gt/*.png                 ← excluded by .gitignore
```
