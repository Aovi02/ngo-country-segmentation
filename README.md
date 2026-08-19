# Data-Driven NGO Aid Prioritization

### Segmenting 167 countries with unsupervised learning to support transparent, evidence-led humanitarian screening

![Python](https://img.shields.io/badge/Python-3.12-3776AB?logo=python&logoColor=white)
![Jupyter](https://img.shields.io/badge/Jupyter-Notebook-F37626?logo=jupyter&logoColor=white)
![scikit-learn](https://img.shields.io/badge/scikit--learn-Clustering-F7931E?logo=scikitlearn&logoColor=white)

This project builds an end-to-end unsupervised machine-learning workflow for an NGO deciding where deeper aid assessment may be most valuable. It converts nine socioeconomic and health indicators into four interpretable country segments, compares three clustering families, and translates the selected model into an actionable—but deliberately non-prescriptive—screening framework.

The work demonstrates more than fitting a clustering algorithm: it tests whether apparently strong metrics produce useful groups, profiles the result in real-world units, checks sensitivity to preprocessing choices, and treats humanitarian allocation as a decision that requires context beyond a model.

> **Important:** The dataset does not include a reference year or source provenance. Findings describe this supplied data snapshot and must not be interpreted as current country conditions or as a final funding recommendation.

## Executive summary

- **167 countries** were assessed across **9 development, economic, and health indicators**. The data contains no missing values, duplicate rows, or duplicate country names.
- A **four-cluster KMeans model** was selected as the most useful stakeholder-facing solution (`random_state=42`, random initialization, `n_init=10`, `max_iter=300`).
- The **High Aid Priority** segment contains **48 countries** and shows the strongest combined signal of humanitarian need: average child mortality of **92.4 per 1,000 live births**, income of **3,938**, GDP per capita of **1,903**, life expectancy of **59.3 years**, and fertility of **5.0 births per woman**.
- Compared with the Moderate Aid Priority segment, the high-priority group has approximately **4.5× higher child mortality**, **70% lower income**, **73% lower GDP per capita**, and **13.7 fewer years of life expectancy**.
- **Haiti, Sierra Leone, Chad, the Central African Republic, and Mali** have the highest child-mortality values within the model's high-priority segment and therefore emerge as candidates for immediate contextual review—not automatic allocation.
- **Luxembourg, Malta, and Singapore** form a three-country specialist-review cluster. Their unusually high trade ratios make them statistically distinct, but their high income and life expectancy show why statistical outliers must not automatically be treated as aid priorities.
- The first principal component explains **46.0%** of variance and is dominated by life expectancy, child mortality, fertility, income, and GDP per capita—evidence that a broad development axis is the dataset's main source of separation. The first four components explain **87.2%**.

## Segment profiles

The labels below are assigned after clustering from each group's mean socioeconomic profile. Values are segment means in the original data units.

| Segment | Countries | Child mortality | Income | GDP per capita | Life expectancy | Fertility | Interpretation |
|---|---:|---:|---:|---:|---:|---:|---|
| **High Aid Priority** | 48 | 92.4 | 3,938 | 1,903 | 59.3 | 5.0 | Strongest combined signal of low resources and adverse health outcomes |
| **Moderate Aid Priority** | 87 | 20.7 | 13,275 | 7,162 | 73.1 | 2.3 | Broad middle-development group; country-level context remains important |
| **Lower Aid Priority** | 29 | 5.0 | 45,762 | 44,066 | 80.4 | 1.8 | High-income, high-life-expectancy group with lower observed humanitarian need |
| **Specialized Review Group** | 3 | 4.1 | 64,033 | 57,567 | 81.4 | 1.4 | Wealthy trade hubs separated by extreme export/import ratios, not aid need |

Child mortality is measured per 1,000 live births; fertility is births per woman. Income and GDP-per-capita units follow the supplied data dictionary, which does not specify currency or price year.

## Why KMeans was selected

The highest internal score was not accepted blindly. Average-linkage hierarchical clustering achieved a silhouette score of **0.630**, but placed **166 countries in one cluster and only 1 in the other**. That is mathematically separated yet operationally unhelpful.

| Candidate | Configuration | Silhouette ↑ | Calinski–Harabasz ↑ | Davies–Bouldin ↓ | Cluster sizes | Decision |
|---|---|---:|---:|---:|---|---|
| **KMeans** | 4 clusters | **0.304** | **62.211** | **1.042** | 48 / 87 / 3 / 29 | Selected for transparent, useful segmentation |
| Hierarchical | 2 clusters, average/Euclidean | 0.630 | 11.391 | 0.264 | 166 / 1 | Rejected as severely imbalanced |
| Gaussian mixture | 4 components, full covariance | 0.175 | 40.346 | 1.577 | 5 / 54 / 72 / 36 | More flexible, but weaker separation and interpretability |

This selection balances internal validation with cluster balance, interpretability, and the NGO's intended use. The three-country KMeans cluster is retained because it captures a coherent structural pattern—trade-intensive, wealthy economies—rather than an arbitrary singleton.

## Analytical workflow

```text
Data validation
    ↓
EDA: distributions, outliers, correlations
    ↓
Standardization + alternative feature sets
    ↓
PCA for structure and visualization
    ↓
KMeans, hierarchical clustering, and GMM tuning
    ↓
Metric, balance, and interpretability comparison
    ↓
Cluster profiling and NGO-facing segment labels
    ↓
Robustness checks and responsible-use recommendations
```

Key design choices:

- **Preserve meaningful outliers:** extreme countries can be central to humanitarian screening, so they were investigated rather than removed automatically.
- **Standardize all model inputs:** the indicators use different scales and units.
- **Compare multiple model families:** KMeans provides a baseline, hierarchical clustering exposes nested structure, and Gaussian mixtures allow probabilistic, non-spherical groups.
- **Use PCA as an explanatory tool:** clustering is evaluated separately in the scaled feature space; PCA supports dimensional interpretation and visual inspection.
- **Test preprocessing sensitivity:** the selected four-segment structure remains plausible with log-adjusted economic features, engineered per-capita value features, and a four-component PCA representation. The PCA variant improves silhouette from approximately **0.302 to 0.355**.

## Dataset

The supplied dataset contains one identifier and nine numerical features:

| Feature | Meaning |
|---|---|
| `country` | Country name; retained for interpretation, excluded from modeling |
| `child_mort` | Deaths of children under five per 1,000 live births |
| `exports` | Exports as a percentage of GDP per capita |
| `health` | Health spending as a percentage of GDP per capita |
| `imports` | Imports as a percentage of GDP per capita |
| `income` | Net income per person |
| `inflation` | Annual growth rate of total GDP |
| `life_expec` | Expected lifespan under current mortality patterns |
| `total_fer` | Expected births per woman under current fertility rates |
| `gdpp` | GDP per capita |

## Repository structure

```text
.
├── Analysis.ipynb           # Main analysis notebook
├── Analysis.executed.ipynb  # Notebook snapshot with executed outputs
├── Country-data.csv         # Country-level input data (167 rows)
├── data-dictionary.csv      # Feature definitions
└── README.md                # Project overview and findings
```

## Run the analysis

Python **3.12** was used for the executed notebook.

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install jupyter numpy pandas matplotlib seaborn scipy scikit-learn
jupyter lab Analysis.ipynb
```

Run the notebook from top to bottom with the CSV files in the repository root. To review the results without rerunning the analysis, open [`Analysis.executed.ipynb`](Analysis.executed.ipynb).

## Responsible interpretation

This model is a **screening and exploration tool**, not an automated aid-allocation system. Country averages can hide local inequality, and the dataset omits direct measures of poverty, conflict, governance, education, food security, climate exposure, program cost, access, and NGO operating constraints. It is also cross-sectional, so it cannot distinguish improvement from deterioration.

Before acting on the results, an NGO should:

1. Verify the dataset's source, year, units, and collection methodology.
2. Refresh each indicator from authoritative, current sources.
3. Add conflict, poverty, food-security, climate-risk, and operational-feasibility data.
4. Review high-priority countries individually with regional experts and local partners.
5. Use subnational and time-series analysis before deciding how funds should be allocated.

## Features of the study

- End-to-end exploratory data analysis and data-quality assessment
- Feature engineering, scaling, and sensitivity analysis
- PCA interpretation and dimensionality reduction
- KMeans, agglomerative clustering, and Gaussian mixture modeling
- Hyperparameter search with multiple unsupervised validation metrics
- Model selection grounded in stakeholder usefulness rather than a single score
- Data visualization with Matplotlib and Seaborn
- Translation of technical results into responsible decision support

## Potential extensions

- Package preprocessing and inference into a reusable scikit-learn pipeline
- Add cluster-stability testing through bootstrapping or resampling
- Quantify membership uncertainty and examine borderline countries
- Incorporate current World Bank, WHO, UNDP, and humanitarian-risk indicators
- Build a stakeholder dashboard with adjustable indicator weights and country drill-downs
- Track movement between segments over time using longitudinal data

