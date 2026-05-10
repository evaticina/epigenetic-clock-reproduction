# Public results (lightweight)

This folder holds **small, publication-style outputs** (figures and summary tables) so they can be browsed on GitHub without bloating the repository.

**What is here:** PNG figures and compact CSV metric tables produced by the analysis notebooks.

**What is intentionally excluded:** Raw GEO downloads, full methylation matrices, processed feature matrices, large intermediate dumps, compressed arrays, caches, and other bulky generated artifacts stay out of version control (see the root `.gitignore`).

**Regeneration:** Run the notebooks in order (`01_data_loading.ipynb` → `02_model_training.ipynb` → `03_biological_interpretation.ipynb`). They write the full artifact set under `results/`. When updating this repository’s public snapshot, copy the refreshed lightweight files from `results/` into `results/public/` as documented in the root README.
