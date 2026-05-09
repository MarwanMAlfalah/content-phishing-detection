# Content-Based Phishing Detection

[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.20099937.svg)](https://doi.org/10.5281/zenodo.20099937)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Python 3.10+](https://img.shields.io/badge/python-3.10+-blue.svg)](https://www.python.org/downloads/)

This repository contains the code, dataset pipeline, and full results for the paper:

> **Beyond URL Analysis: Evaluating Content-Based Phishing Detection Using HTML Structural Features**
> Y. A. Al-Haj, M. M. Al-Falah, N. Mohammed, E. S. Gebel, A. M. H. Obaid
> *Journal of Artificial Intelligence Topics and Applications (AITA)*, 2026.

The study evaluates how well phishing webpages can be classified using only features extracted from HTML content, with no URL lexical properties, reputation lookups, or network signals. Seven supervised classifiers are trained on a balanced 40,000-page corpus and analyzed for feature importance, class-specific behavior, and failure modes.

**Archived release:** [10.5281/zenodo.20099937](https://doi.org/10.5281/zenodo.20099937)

---

## Repository Contents

A complete reproducible pipeline that:

1. Collects phishing URLs from PhishTank and legitimate domains from Tranco
2. Retrieves each page through live HTTP fetch
3. Extracts a 15-dimensional feature vector from the HTML source
4. Trains and evaluates seven classifiers under stratified 5-fold cross-validation
5. Generates interpretability artifacts: permutation importance, SHAP analysis, ablation study, and qualitative error analysis

The collection pipeline includes incremental checkpointing, allowing the run to resume from any interruption without losing progress.

---

## Quick Start

```bash
git clone https://github.com/MarwanMAlfalah/content-phishing-detection.git
cd content-phishing-detection

python -m venv venv
source venv/bin/activate          # macOS / Linux
# venv\Scripts\activate           # Windows

pip install -r requirements.txt
jupyter notebook Content_Based_Phishing_Detection_v6_local.ipynb
```

Then select `Kernel → Restart & Run All`. Full collection takes 10–25 hours depending on network conditions. To skip collection and reproduce only the analysis on the published dataset, run cells 15 onward; this completes in approximately 10 minutes.

---

## Data Sources

The pipeline supports a fallback chain for phishing URL acquisition:

1. **PhishTank** (preferred) — community-verified phishing feed. If you have a working account, place `online-valid.csv` in the project root and the notebook will use it automatically.
2. **OpenPhish** — free community feed
3. **PhishStats** — phishing URL statistics
4. **URLhaus** — malicious URL database maintained by abuse.ch

Legitimate domains are sourced from Tranco's research-grade top-domain ranking. The full collection methodology, including timeout values, deduplication strategy, and library versions, is documented in Section III.B of the paper.

---

## Outputs

A complete run populates `results/` with the following:

| File | Paper Reference |
|------|-----------------|
| `results_cv_mean_std.csv` | Table 4 — Cross-validation performance |
| `aggregated_confusion_metrics.csv` | Table 5 — Aggregated confusion matrices |
| `rf_permutation_importance.csv` | Table 6 — Feature importance ranking |
| `ablation_study.csv` | Table 7 — Ablation results |
| `figure1_roc_curves.{png,pdf}` | Figure 1 — ROC curves |
| `figure2_shap_global.{png,pdf}` | Figure 2 — Global SHAP summary |
| `figure3a_shap_phishing_class.{png,pdf}` | Figure 3a — SHAP for phishing class |
| `figure3b_shap_legitimate_class.{png,pdf}` | Figure 3b — SHAP for legitimate class |
| `figure4_shap_dependence_top3.png` | Figure 4 — Dependence plots |
| `figure3_error_profiles.png` | Figure 5 — Error feature profiles |
| `all_results.xlsx` | Combined Excel workbook |
| `metadata.json` | Run configuration and package versions |

---

## Feature Set

All 15 features are derived from parsed HTML using BeautifulSoup. URL strings are not used at any stage of feature extraction or model training.

**Textual** (4): `text_len`, `word_count`, `title_len`, `keyword_hits`
**Structural** (8): `n_scripts`, `n_images`, `n_forms`, `n_inputs`, `n_iframes`, `has_password_field`, `has_submit_field`, `has_email_field`
**Hyperlink** (3): `n_links`, `n_external_links`, `external_link_ratio`

External links are identified by comparing registered domains using `tldextract` rather than raw netloc strings. The phishing keyword list (`verify`, `login`, `password`, `update`, `bank`, `account`, `confirm`, `secure`) is counted as distinct presence per page, not occurrence frequency.

Complete feature definitions are provided in Table 3 of the paper.

---

## Headline Results

Performance on the balanced 40,000-page corpus (20,000 phishing + 20,000 legitimate), stratified 5-fold cross-validation:

| Model | F1 | AUC | FNR | FPR |
|-------|-----|------|------|------|
| **Random Forest** | **0.940 ± 0.002** | **0.983 ± 0.001** | 0.065 | 0.053 |
| Decision Tree | 0.921 ± 0.002 | 0.929 ± 0.002 | 0.070 | 0.089 |
| KNN | 0.913 ± 0.004 | 0.956 ± 0.002 | 0.101 | 0.071 |
| MLP | 0.908 ± 0.002 | 0.970 ± 0.001 | 0.057 | 0.135 |
| SVM-RBF | 0.894 ± 0.004 | 0.945 ± 0.003 | 0.058 | 0.164 |
| AdaBoost | 0.892 ± 0.004 | 0.957 ± 0.003 | 0.051 | 0.179 |
| Gaussian NB | 0.838 ± 0.003 | 0.885 ± 0.004 | 0.027 | 0.349 |

Beyond the headline numbers, the paper's central finding is that predictive signal concentrates in four features (`n_links`, `n_scripts`, `n_images`, `title_len`), with three intuitively-relevant binary features (`has_password_field`, `has_submit_field`, `has_email_field`) contributing marginally. A 5-feature subset retains F1 = 0.926, only 1.4 points below the full model. The paper analyzes why.

---

## Reproducibility

The random seed is fixed at `42` throughout. Standardization is performed inside the cross-validation loop using `ColumnTransformer` with parameters estimated from the training fold only, preventing information leakage. The library versions used in the published results:

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

These are pinned with minimum version constraints in `requirements.txt`. Exact versions for any given run are recorded in `results/metadata.json`.

The frozen v1.0.0 release archived on Zenodo ([10.5281/zenodo.20099937](https://doi.org/10.5281/zenodo.20099937)) corresponds to the exact code state used to produce the results reported in the paper.

---

## Practical Notes

A few operational details worth highlighting:

- **macOS sleep prevention.** For long collection runs, execute `caffeinate -dims &` in a separate terminal before starting the notebook to prevent the system from sleeping mid-collection.
- **Tranco caching.** The Tranco library caches domain lists in `.tranco_cache/`. Initial fetches require network access; subsequent calls are served from cache.
- **Phishing URL volatility.** PhishTank URLs are short-lived; expect 30–70% fetch success rates depending on feed recency. The pipeline applies 5x oversampling to compensate.
- **Checkpointing.** The pipeline saves progress every 50 successful pages. If the kernel terminates unexpectedly, re-running cells 11–13 resumes from the last checkpoint.
- **Hardware.** Results in the paper were generated on a MacBook Air M4 (Apple Silicon). The pipeline is portable across Intel and Linux platforms with no architecture-specific code.

---

## Scope and Limitations

This study evaluates content-based phishing detection within a controlled experimental setting. The following design choices and constraints are explicitly noted:

- The dataset is balanced 50:50 by design to isolate feature-level analysis from class-imbalance effects. This does not reflect the natural class distribution observed in production environments.
- Hyperparameters are set to scikit-learn defaults to establish a baseline that isolates the contribution of the feature representation. Tuning is identified as future work in Section VI of the paper.
- Training and test data are drawn from the same collection window. Temporal validation, cross-dataset transfer, and adversarial robustness evaluation are not within the scope of this study.
- Feature extraction operates on static HTML. Pages whose primary content is rendered through JavaScript after page load are not fully characterized.

The paper's Section VI provides a complete discussion of limitations and outlines specific directions for follow-up work.

---

## Citation

If this code or dataset contributes to your research, please cite both the paper and the archived software release:

```bibtex
@article{alhaj2026beyond,
  title   = {Beyond URL Analysis: Evaluating Content-Based Phishing Detection
             Using HTML Structural Features},
  author  = {Al-Haj, Yousif A. and Al-Falah, Marwan M. and Mohammed, Naif and
             Gebel, E. S. and Obaid, Abdulrahman M. H.},
  journal = {Journal of Artificial Intelligence Topics and Applications (AITA)},
  year    = {2026}
}

@software{alfalah2026phishing_software,
  author       = {Al-Falah, Marwan M.},
  title        = {{MarwanMAlfalah/content-phishing-detection: Initial release}},
  month        = may,
  year         = 2026,
  publisher    = {Zenodo},
  version      = {v1.0.0},
  doi          = {10.5281/zenodo.20099937},
  url          = {https://doi.org/10.5281/zenodo.20099937}
}
```

---

## License

Released under the MIT License. See [LICENSE](LICENSE) for full terms.

---

## Contact

For questions related to the methodology or experimental design, contact the corresponding author Al-Falah, Marwan M. at `alfalah.m@edu.spbstu.ru`. For implementation issues, please open a GitHub issue so that responses are visible to all users.
