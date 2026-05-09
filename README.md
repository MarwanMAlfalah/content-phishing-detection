# content-phishing-detection
=======
# Content-Based Phishing Detection (HTML-Only)

This repository contains the code, data pipeline, and results for the paper:

> **Beyond URL Analysis: Evaluating Content-Based Phishing Detection Using HTML Structural Features**
> Y. A. Al-Haj, M. M. Al-Falah, N. Mohammed, E. S. Gebel, A. M. H. Obaid
> *Journal of Artificial Intelligence Topics and Applications (AITA)*, 2026.

The short version: we asked how far you can get classifying phishing pages using only what you can extract from the HTML — no URL strings, no reputation lookups, no network telemetry. The answer turned out to be more interesting than we expected.

---

## What's in here

A complete, reproducible pipeline that:

1. Pulls phishing URLs from PhishTank and legitimate domains from Tranco
2. Fetches each page live over HTTP
3. Extracts 15 features from the raw HTML
4. Trains and evaluates seven classifiers under stratified 5-fold cross-validation
5. Produces interpretability artifacts (permutation importance, SHAP, ablation, error analysis)

Everything lives in a single Jupyter notebook with checkpointing built in, so if your run gets interrupted at hour 14 of 20, it picks up where it left off.

---

## Quick start

```bash
git clone https://github.com/MarwanMAlfalah/content-phishing-detection.git
cd content-phishing-detection

python -m venv venv
source venv/bin/activate          # macOS / Linux
# venv\Scripts\activate           # Windows

pip install -r requirements.txt
jupyter notebook Content_Based_Phishing_Detection_v6_local.ipynb
```

Then `Kernel → Restart & Run All`. Go make coffee. The full collection takes 10–25 hours depending on your network.

If you want to skip the data collection and just rerun the analysis on our results, the dataset CSV is in `results/phish_tranco_dataset.csv`. Run cells 15 onward and you'll have everything in about 10 minutes.

---

## A note on PhishTank

PhishTank temporarily disabled new account registration during our collection window. The notebook handles this with a fallback chain: it tries PhishTank's auto-download first, then OpenPhish, PhishStats, and URLhaus in order. If you have PhishTank's `online-valid.csv` from a working account, drop it in the project root and the notebook will pick it up automatically — that's the cleanest source.

If none of those work for you, you'll get a useful error pointing to the manual download steps.

---

## What you get

After a complete run, the `results/` folder contains:

| File | Maps to |
|------|---------|
| `results_cv_mean_std.csv` | Table 4 — 5-fold CV performance |
| `aggregated_confusion_metrics.csv` | Table 5 — confusion matrices |
| `rf_permutation_importance.csv` | Table 6 — feature ranking |
| `ablation_study.csv` | Table 7 — reduced feature subsets |
| `figure1_roc_curves.{png,pdf}` | Figure 1 |
| `figure2_shap_global.{png,pdf}` | Figure 2 |
| `figure3a_shap_phishing_class.{png,pdf}` | Figure 3a |
| `figure3b_shap_legitimate_class.{png,pdf}` | Figure 3b |
| `figure4_shap_dependence_top3.png` | Figure 4 |
| `figure3_error_profiles.png` | Figure 5 |
| `all_results.xlsx` | Everything in one Excel workbook |
| `metadata.json` | Exact package versions, seeds, dataset stats |

You'll also see `_checkpoint.csv` and `errors.csv` during the run. The first is the resume state; the second is a log of fetch failures that's useful when something goes wrong.

---

## The features

All 15 features come from parsing the HTML with BeautifulSoup. No URL strings touch the model.

**Text** (4): `text_len`, `word_count`, `title_len`, `keyword_hits`
**Structure** (8): `n_scripts`, `n_images`, `n_forms`, `n_inputs`, `n_iframes`, `has_password_field`, `has_submit_field`, `has_email_field`
**Hyperlinks** (3): `n_links`, `n_external_links`, `external_link_ratio`

External links are detected by comparing registered domains via `tldextract`, not the raw netloc. The keyword list is `verify`, `login`, `password`, `update`, `bank`, `account`, `confirm`, `secure` — counted as distinct presence, not occurrence frequency.

Full definitions are in Table 3 of the paper.

---

## Headline numbers

From the latest run on a balanced 40,000-page corpus:

| Model | F1 | AUC | FNR | FPR |
|-------|-----|------|------|------|
| **Random Forest** | **0.940** ± 0.002 | **0.983** ± 0.001 | 0.065 | 0.053 |
| Decision Tree | 0.921 ± 0.002 | 0.929 ± 0.002 | 0.070 | 0.089 |
| KNN | 0.913 ± 0.004 | 0.956 ± 0.002 | 0.101 | 0.071 |
| MLP | 0.908 ± 0.002 | 0.970 ± 0.001 | 0.057 | 0.135 |
| SVM-RBF | 0.894 ± 0.004 | 0.945 ± 0.003 | 0.058 | 0.164 |
| AdaBoost | 0.892 ± 0.004 | 0.957 ± 0.003 | 0.051 | 0.179 |
| Gaussian NB | 0.838 ± 0.003 | 0.885 ± 0.004 | 0.027 | 0.349 |

The interesting bit isn't that Random Forest wins — that's expected on tabular data this size. The interesting bit is that four features (`n_links`, `n_scripts`, `n_images`, `title_len`) carry most of the signal, while three intuitively-promising binary features (`has_password_field`, `has_submit_field`, `has_email_field`) contribute almost nothing. A 5-feature subset reaches F1 = 0.926, only 1.4 points below the full model.

The paper digs into why.

---

## Reproducibility

Random seed is fixed at `42` throughout. All standardization happens inside the cross-validation loop (training fold only) using `ColumnTransformer` — no leakage. Versions used:

```
Python 3.11
scikit-learn 1.8
numpy 2.4
pandas 3.0
beautifulsoup4 4.12
requests 2.31
tldextract 5.1
shap 0.51
```

These are pinned in `requirements.txt` with minimum versions. The exact versions used at runtime are recorded in `results/metadata.json` after each run.

---

## Tips from running this in practice

A few things we learned the hard way:

- **On macOS, run `caffeinate -dims &` in another terminal before starting collection.** Otherwise your laptop will sleep around hour 4 and you'll be sad.
- **Tranco caches its list locally** under `.tranco_cache/`. The first call fetches; subsequent calls are instant. Don't delete this folder unless you want to re-download.
- **PhishTank URLs expire fast.** Expect 30–70% fetch success depending on how recent the feed is. The pipeline accounts for this with a 5x oversampling ratio.
- **Checkpoints save every 50 successful pages.** If the kernel dies, just re-run cells 11–13 and it'll pick up.
- **The paper's results were generated on a MacBook Air M4** (Apple Silicon). Everything works on Intel and Linux too, no Apple-specific code.

---

## Limitations (honest version)

This isn't a deployment-ready phishing detector. A few things to keep in mind:

- The 50:50 class balance is an artifact of the experimental design, not the wild distribution
- We didn't tune hyperparameters — defaults only, by choice
- No temporal split: train and test pages were collected in the same window
- No cross-dataset evaluation
- No adversarial robustness testing
- No JavaScript rendering — we parse static HTML only

The paper has a full Limitations section that goes into this in more detail. If you're using this code as a starting point for production work, please read Section VI before drawing conclusions.

---

## Citation

If you use this code, please cite the paper:

```bibtex
@article{alhaj2026beyond,
  title   = {Beyond URL Analysis: Evaluating Content-Based Phishing Detection
             Using HTML Structural Features},
  author  = {Al-Haj, Yousif A. and Al-Falah, Marwan M. and Mohammed, Naif and
             Gebel, E. S. and Obaid, Abdulrahman M. H.},
  journal = {Journal of Artificial Intelligence Topics and Applications (AITA)},
  year    = {2026}
}
```

---

## License

MIT — see [LICENSE](LICENSE). Use it, fork it, build on it. A citation if you publish based on it would be appreciated.

---