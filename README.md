# The Spanish Gender Wage Gap: Causal Estimation, Simulation Evidence, and Lifetime Economic Consequences 

**Master's Thesis — MESIO UPC-UB (Master in Statistics and Operations Research)**  
**Author:** Julia Anglada Lomaeva  
**Faculty:** FME — Facultat de Matemàtiques i Estadística, Universitat Politècnica de Catalunya  
**Year:** 2025–2026

---

## Overview

This repository contains all R scripts, Rmd notebooks, and outputs for my Master's in Statistics Thesis on the gender wage gap in Spain using EES 2022 microdata.

The empirical strategy moves from description to causal estimation in four stages:

1. **OLS** (Ordinary Least Squares) — adjusted baseline gap
2. **Oaxaca-Blinder decomposition** — separates composition vs. pricing effects
3. **Double Machine Learning (DML)** — causal estimate under non-linear confounding
4. **Monte Carlo life-cycle simulation** — translates the hourly penalty into lifetime NPV

**Key results:**
- Raw gap: **16.64%** (men €18.99/hr vs women €15.83/hr, EES 2022)
- OLS Spec 2 adjusted penalty: **−13.9%**
- DML estimate (RF nuisance): **−16.24%** (θ̂ = −0.1772, SE = 0.0021)
- OB unexplained component: **121.7%** of the total gap — stable across all reference structures
- Median lifetime income gap (NPV, r = 2%): **€175,957** ≈ 74 months of average salary

---

## Data

This analysis uses the **Encuesta de Estructura Salarial (EES) 2022**, published by the Instituto Nacional de Estadística (INE).

**The microdata file (`EES_2022.RData`) is NOT included in this repository** — it is subject to INE's statistical confidentiality provisions (Ley 12/1989) and cannot be redistributed. To replicate this analysis:

1. Download the EES 2022 microdata from the [INE website](https://www.ine.es/dyngs/INEbase/es/operacion.htm?c=Estadistica_C&cid=1254736177025)
2. Place `EES_2022.RData` in `data/raw/`
3. Place `dr_EES_2022.xlsx` (the official INE design record with classification tables) in `data/mappings/`
4. Run `scripts/00_preprocessing.R`

---

## Repository Structure

```
tfm-gender-wage-gap-spain/
├── README.md
├── .gitignore
├── data/
│   ├── raw/              # EES_2022.RData — NOT included (INE license)
│   └── mappings/         # dr_EES_2022.xlsx — classification mapping tables
├── scripts/
│   ├── 00_preprocessing.R          # Data loading, decoding, feature engineering
│   ├── 01_EDA.Rmd                  # Exploratory data analysis
│   ├── 02_OLS_models.Rmd           # OLS specifications + HC3 robust SEs
│   ├── 03_oaxaca_blinder.Rmd       # Oaxaca-Blinder decomposition
│   ├── 04_DML.Rmd                  # Double Machine Learning (RF + Lasso)
│   ├── 04b_simulation_study_full.Rmd  # Simulation study: OLS vs DML under DGP 1 and DGP 2 v1–v6
│   ├── 05_Bootstrap Stability and Sensitivity Analysis.Rmd  # Bootstrap stability
│   └── 06_monte_carlo_lifecycle.Rmd   # Monte Carlo NPV life-cycle simulation
└── outputs/
    └── figures/          # All .png figures used in the thesis
```

---

## How to Reproduce

### Requirements

- R ≥ 4.3.0
- Key packages: `tidyverse`, `DoubleML`, `mlr3`, `oaxaca`, `sandwich`, `lmtest`, `ineq`, `ranger`, `glmnet`, `rmarkdown`

Install all at once:

```r
install.packages(c(
  "tidyverse", "DoubleML", "mlr3", "mlr3learners",
  "oaxaca", "sandwich", "lmtest", "ineq",
  "ranger", "glmnet", "rmarkdown", "knitr"
))
```

### Running order

```r
# Step 1 — always run first in a new session
source("scripts/00_preprocessing.R")

# Step 2 — render each analysis script
rmarkdown::render("scripts/01_EDA.Rmd",          envir = globalenv())
rmarkdown::render("scripts/02_OLS_models.Rmd",   envir = globalenv())
rmarkdown::render("scripts/03_oaxaca_blinder.Rmd", envir = globalenv())
rmarkdown::render("scripts/04_DML.Rmd",          envir = globalenv())
rmarkdown::render("scripts/04b_simulation_study_full.Rmd", envir = globalenv())
rmarkdown::render("scripts/05_Bootstrap Stability and Sensitivity Analysis.Rmd", envir = globalenv())
rmarkdown::render("scripts/06_monte_carlo_lifecycle.Rmd", envir = globalenv())
```

> **Note:** Each Rmd includes an auto-load block that calls `00_preprocessing.R` automatically if `df_main` is not found in the environment. Using `envir = globalenv()` is still recommended to ensure all objects are shared correctly across scripts.

---

## Methodology Summary

| Method | Question answered | Key result |
|--------|-------------------|------------|
| OLS (3 specs) | How large is the adjusted gap? | −13.9% (Spec 2) |
| Oaxaca-Blinder | Where does the gap come from? | 121.7% unexplained |
| DML (RF + Lasso) | Is OLS correctly estimating the penalty? | −16.24% (RF), −16.67% (Lasso) |
| Simulation study | When does DML outperform OLS? | OLS: 0% coverage under DGP 2 v6; DML: 78.8% |
| Bootstrap (B=200) | Is the DML result sample-stable? | 100% of samples give θ̂ < 0 |
| MC life-cycle | What does the gap cost over a career? | Median NPV €175,957 |

---

## Key References

- Chernozhukov, V. et al. (2018). Double/Debiased Machine Learning. *Econometrics Journal* 21(1). doi:10.1111/ectj.12097
- Oaxaca, R. (1973). Male-Female Wage Differentials in Urban Labor Markets. *IER* 14(3). doi:10.2307/2525981
- Blinder, A.S. (1973). Wage Discrimination. *JHR* 8(4). doi:10.2307/144855
- Mincer, J. (1974). *Schooling, Experience, and Earnings*. NBER.
- Goldin, C. (2014). A Grand Gender Convergence. *AER* 104(4). doi:10.1257/aer.104.4.1091
- De la Rica, S., Dolado, J.J. & Llorens, V. (2008). Ceilings or Floors? *J. Population Econ.* 21(3). doi:10.1007/s00148-006-0128-1
- INE (2022). *Encuesta de Estructura Salarial 2022*. Instituto Nacional de Estadística.

---

## License

The code in this repository is available under the MIT License. The thesis document and figures are © Julia Anglada Lomaeva, 2026. The EES 2022 microdata is © INE and subject to its own license terms — see the INE website for conditions of use.

---

## Contact

Julia Anglada Lomaeva  
MESIO UPC-UB, 2025–2026
