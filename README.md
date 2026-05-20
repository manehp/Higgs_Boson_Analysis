# A Machine-Learning Analysis of the Higgs Boson in the H → ZZ\* → 4ℓ Channel

Capstone project, BS in Data Science, American University of Armenia.
Author: Mane Papoyan. Supervisor: Houry Keoshkerian.

## Project Objective

Test whether a standalone machine-learning classifier, trained only on
simulated H → ZZ\* → 4ℓ events, can identify the Higgs boson when applied
directly to the 1,279 real ATLAS events in the 2025 Open Data release at
√s = 13 TeV, ∫L = 36.6 fb⁻¹.

Seven classifiers (XGBoost, LightGBM, Random Forest, Keras MLP, Logistic
Regression, QDA, Gaussian Naive Bayes) are trained under a common 5-fold
stratified cross-validation protocol on 20 physics-motivated features, then
applied to real data and evaluated with two independent background-estimation
methods (MC prediction and data-driven sideband extrapolation). XGBoost and
LightGBM reach evidence-level significance (Z ≈ 4.7σ and 5.1σ under the
sideband method).

## Repository Layout

```
capstone_project/
├── CapstonePaper_ManePapoyan/
│   ├── main.tex
│   ├── references.bib
│   ├── figures/
│   └── CapstonePaper_ManePapoyan.pdf
├── code/
│   └── Higgs_Analysis.ipynb
├── data/
│   └── Data.md
└── README.md
```

* `CapstonePaper_ManePapoyan/` — LaTeX source, bibliography, generated
  figures, and compiled PDF of the paper.
* `code/Higgs_Analysis.ipynb` — full analysis notebook (data loading,
  preselection, Z₁/Z₂ reconstruction, feature engineering, 7-classifier
  training, real-data evaluation, significance computation, plots).
* `data/Data.md` — data sources, retrieval instructions, preprocessing steps,
  and variable definitions. **Datasets are not redistributed**; they stream
  from the ATLAS Open Data servers at runtime via `atlasopenmagic`.
* `README.md` — this file.

## Required Software and Libraries

* **Python** 3.11
* **Platform** — developed and tested on Google Colab (single CPU runtime,
  no GPU). End-to-end run ≈ 127 minutes.

Python packages:

| Package           | Purpose                                                    |
| ----------------- | ---------------------------------------------------------- |
| `uproot`          | Read ROOT files                                            |
| `awkward`         | Nested / variable-length arrays                            |
| `vector`          | Four-momentum operations and Lorentz boosts                |
| `atlasopenmagic`  | ATLAS Open Data access + caching                           |
| `numpy`, `pandas` | Core numerics and tables                                   |
| `scipy`           | `scipy.stats.norm` for p-values                            |
| `scikit-learn`    | Random Forest, Logistic Regression, QDA, GaussianNB, CV utils, scaler, metrics |
| `xgboost`         | XGBoost classifier                                         |
| `lightgbm`        | LightGBM classifier                                        |
| `tensorflow`      | Keras MLP                                                  |
| `matplotlib`      | Plotting                                                   |
| `seaborn`         | Correlation-matrix heatmap                                 |

Install everything in one line:

```bash
pip install uproot awkward vector atlasopenmagic xgboost lightgbm tensorflow \
            scikit-learn scipy numpy pandas matplotlib seaborn
```

(The notebook's first cell runs this automatically inside Colab.)

## Steps to Reproduce

1. **Clone the repository.**

   ```bash
   git clone https://github.com/manehp/Higgs_Boson_Analysis
   cd Higgs_Boson_Analysis
   ```

2. **Open the notebook.** Either upload `code/Higgs_Analysis.ipynb` to
   Google Colab (recommended) or open it in a local Jupyter environment.

3. **Read `data/Data.md`** for the exact list of dataset IDs (DIDs) used,
   the cross-section/luminosity weighting formula, and variable definitions.
   No manual data download is required — the notebook streams the ROOT files
   from the ATLAS Open Data servers via `atlasopenmagic` on first run, then
   caches them locally.

4. **Run the notebook top to bottom.** Random seeds are pinned to 42
   throughout (fold partitioning, model initialisation, train/val splits) so
   a fresh run reproduces every reported number to within < 0.005 in AUC
   and ~3 % in significance. Training all seven classifiers under the
   5-fold protocol takes ≈ 127 minutes on a Colab CPU runtime.

5. **Inspect outputs.** The notebook prints all yields, AUCs, accuracies,
   and significances, and saves every figure as a PNG file in the working
   directory.

## How to Run the Code

The entire analysis lives in a single notebook organised into 14 sections,
in execution order:

| §  | Purpose                                                                                              |
| -- | ---------------------------------------------------------------------------------------------------- |
| 1  | Imports and environment setup                                                                        |
| 2  | Data and simulated samples — `atlasopenmagic` configuration, dataset definitions                     |
| 3  | Preselection cuts (trigger, lepton ID/iso, SFOS, pT staircase)                                       |
| 4  | m_4ℓ spectrum plot                                                                                   |
| 5  | Z₁/Z₂ reconstruction and 31-feature engineering                                                      |
| 6  | Feature-distribution plots                                                                           |
| 7  | Feature ranking (separation power δ²) and correlation pruning → 20 features                          |
| 8  | Event-yield tables                                                                                   |
| 9  | Shared helpers (significance formula, MC-only no-ML baseline)                                        |
| 10 | XGBoost — grid search, 5-fold OOF, k-fold stability, seed stability, threshold scan                  |
| 11 | LightGBM, Random Forest, Keras MLP, Logistic Regression, QDA, GaussianNB (same protocol)             |
| 12 | Model comparison on MC — accuracy / efficiency / rejection, score distributions, ROC, MC forest plot |
| 13 | Application to real ATLAS data — per-classifier MC-prediction and sideband significance              |
| 14 | Real-data significance forest plot across classifiers                                                |

To re-run a specific block (e.g. only LightGBM on real data), execute §1–§9
first (these build the MC arrays and helpers), then jump to the LightGBM
cell in §13.

## Reproducing the Figures and Tables in the Paper

| Paper item                | Source in `Higgs_Analysis.ipynb`                                  | Output                                          |
| ------------------------- | ----------------------------------------------------------------- | ----------------------------------------------- |
| Fig. 1                    | External (Feynman diagram, not generated by the notebook)         | `feynman_diagram.png`                                              |
| Fig. 2 (m_4ℓ spectrum)    | §4                                                                | `m4l_spectrum.png`                              |
| Fig. 3 (lepton pT)        | §6, `plot_2x2_lepton_pt`                                          | `lepton_pt.png`                                 |
| Fig. 4 (separation power) | §7.3                                                              | `feature_separation.png`                        |
| Fig. 5 (correlation)      | §7.1                                                              | `correlation_matrix.png`                        |
| Fig. 6 (ROC + 95 % CI)    | §12.4                                                             | `roc_curves.png`                                |
| Fig. 7 (OOF score dist.)  | §12.3                                                             | `score_distributions.png`                       |
| Fig. 8 (XGB threshold)    | §10.4                                                             | `xgb_threshold_scan.png`                        |
| Fig. 9 (MC forest plot)   | §12.5                                                             | `mc_significance_forest.png`                    |
| Fig. 10 (data forest plot)| §14                                                               | `realdata_significance_forest.png`              |
| Table I (pT staircase)    | Stated in §3 markdown; cuts implemented in §3 preselection code   | Printed to stdout                                               |
| Table II (yields)         | §8                                                                | Printed to stdout                               |
| Table III (hyperparams)   | §10 (XGB), §11 (other models)                                     | Printed in fit logs                             |
| Table IV (OOF AUC)        | §12.4                                                             | Printed to stdout                               |
| Table V (acc / eff / rej) | §12.1                                                             | Printed to stdout                               |
| Table VI (MC Z)           | §12.5                                                             | Printed to stdout                               |
| Table VII (real-data Z)   | §13 per-classifier blocks                                         | Printed to stdout                               |
| Table VIII (XGB grid)     | §10                                                               | Printed to stdout                               |

All PNG files are written to the notebook's working directory and were copied
into `CapstonePaper_ManePapoyan/figures/` for inclusion in the LaTeX source.

## License and Citation

> M. Papoyan, H. Keoshkerian, *"A Machine-Learning Analysis of the Higgs
> Boson in the H → ZZ\* → 4ℓ Channel"*, BS Capstone, American University
> of Armenia, 2025.

> ATLAS Collaboration, *"ATLAS Open Data: 13 TeV dataset details,"* ATLAS
> Open Data Portal, 2020.
> <https://opendata.atlas.cern/docs/data/for_education/13TeV25_details>
