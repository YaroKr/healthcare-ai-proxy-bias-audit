Auditing Proxy Bias in Healthcare AI
An independent technical replication of the proxy-bias mechanism behind the Optum/UnitedHealth population-health risk algorithm (Obermeyer et al., 2019), tested on a public hospital-readmission dataset.
This repository accompanies an individual Ethical AI audit report. It contains the Google Colab notebook, the generated figures, and the written report.
---
What this project is (and is not)
In 2019, Obermeyer et al. showed in Science that a widely used commercial healthcare algorithm, later attributed to Optum, systematically underestimated the health needs of Black patients. The algorithm never used race as an input. It predicted healthcare cost as a stand-in for healthcare need, and because less money had historically been spent on Black patients' care for reasons of unequal access, cost under-represented their true illness burden.
Optum's data was never released, so this project does not reproduce that algorithm. Instead, it tests whether the same mechanism, a proxy that tracks access rather than need, appears in an independent, public healthcare prediction task: 30-day hospital readmission, using the Diabetes 130-US Hospitals dataset (Strack et al., 2014). Here, prior healthcare utilisation plays the role that cost played for Optum.
The aim is to demonstrate the mechanism is structural, not a one-off flaw in a single system.
---
Key findings
Finding	Result
True 30-day readmission rate (Black vs. White)	8.6% vs. 9.1% — near-equal
Clinical severity at visit (Black vs. White)	12.5 vs. 13.1 — near-equal
Prior utilisation, the proxy (Black vs. White)	26% lower for Black patients
Utilisation per unit of severity	0.036 vs. 0.046 — a 23% shortfall despite comparable need
Disparate impact (Black vs. White), 4/5ths rule	0.775 (LogReg), 0.797 (RF), 0.839 (XGBoost)
Two patients with near-identical real risk and severity are treated differently by the model, because the proxy it relies on is itself shaped by unequal access.
A few points the analysis is careful about:
Model choice is a fairness lever. The same data and features produce disparate impact below the legal 4/5ths threshold for two of three models, but not the third. XGBoost was simultaneously the most accurate and the fairest.
SHAP and LIME disagree, and that matters. SHAP ranks the utilisation proxy as the second most important feature globally, but LIME could not surface it for individual patients, even the highest-utilisation patient in the test set. Population-level bias does not guarantee an individually visible explanation, a real limit on explainability as a safeguard (Krishna et al., 2022).
Small groups are flagged as such. Asian (n=491) and Other (n=1,160) disparate-impact estimates are reported but treated as noisy, not weighted equally with the African American finding (n=12,692).
---
Repository contents
```
.
├── README.md                          # this file
├── Ethical_Artificial_Intelligence_Implementation_IndAssignment.ipynb  # the Colab notebook (full pipeline)
├
└── outputs/
    ├── plots/                         # fairness + proxy-bias figures
    ├── explainability/                # SHAP global + by-group, LIME explanations
    └── tables/                        # metric summaries as CSV
```
---
Running the notebook
The notebook runs end-to-end in Google Colab with no setup beyond the install cell. To run locally instead:
```bash
pip install pandas numpy scikit-learn xgboost fairlearn shap lime matplotlib
```
The dataset downloads automatically in the notebook from a public mirror. The original source is the UCI Machine Learning Repository.
Pipeline steps
Load and clean (deduplicate to one encounter per patient to prevent train/test leakage; remove in-hospital deaths; build the 30-day-readmission target)
Feature engineering and encoding
Train three models (logistic regression, random forest, XGBoost)
Group fairness metrics (demographic parity, equalized odds, disparate impact) via Fairlearn
The proxy-bias check: does utilisation track severity equally across race?
SHAP, global and by-group feature attribution
LIME, matched individual-patient explanations
Results are reproducible with `random_state=42` throughout. Reported model accuracy (~0.67–0.72) and AUROC (~0.60–0.63) are consistent with published benchmarks on this dataset (Emi-Johnson and Nkrumah, 2025); 30-day readmission is a genuinely hard prediction task, and these figures reflect that, not a pipeline fault.
---
Limitations
Utilisation is a proxy for cost, not identical to it; effect sizes here should not be read as estimates of Optum's actual bias.
This is a single dataset from US hospitals, 1999–2008; findings illustrate a mechanism, they do not generalise quantitatively to other settings.
No bias mitigation is applied here; the project audits and diagnoses, it does not attempt to correct.
---
Key references
Obermeyer, Z., Powers, B., Vogeli, C. and Mullainathan, S. (2019) 'Dissecting racial bias in an algorithm used to manage the health of populations', Science, 366(6464), pp. 447–453.
Strack, B. et al. (2014) 'Impact of HbA1c Measurement on Hospital Readmission Rates', BioMed Research International, 2014.
Lundberg, S. and Lee, S.-I. (2017) 'A Unified Approach to Interpreting Model Predictions', NeurIPS.
Ribeiro, M.T., Singh, S. and Guestrin, C. (2016) '"Why Should I Trust You?": Explaining the Predictions of Any Classifier', KDD.
Krishna, S. et al. (2022) 'The Disagreement Problem in Explainable Machine Learning', arXiv:2202.01602.
Full reference list is in the report.
---
A note on authorship
This was produced as an individual university assignment. The analysis, code, and writing are my own; an AI assistant was used for planning, research support, and code debugging, with all outputs independently verified. See the report's AI-use declaration for detail.
Shared as a portfolio piece. If you are reviewing this for the module it was submitted to, please check the relevant academic-conduct guidance before reuse.
