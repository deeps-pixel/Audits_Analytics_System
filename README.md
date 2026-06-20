# Financial Data Quality & Audit Analytics System

Replication of core ACL/IDEA audit analytics procedures in Python, applied to 284,807 real credit card transactions. Findings visualised in a 3-page Power BI dashboard.

Built to demonstrate audit analytics competency without requiring an ACL software license — the same procedures ACL automates (Benford's Law, duplicate detection, risk stratification, stratified sampling) are implemented from scratch and validated before reliance.

---

## What This Does

Takes a raw transactional dataset and runs it through a 6-phase audit analytics pipeline:

1. **Data Inventory** — schema validation, population statistics, diurnal pattern identification
2. **Data Quality Checks** — completeness, exact/near duplicates, outlier analysis, zero-amount detection, round-number test
3. **Benford's Law Analysis** — first-digit test (MAD, chi-square, per-digit Z-scores) across full population, legitimate subgroup, and fraud subgroup; $1.00 sensitivity test
4. **Risk Stratification** — 7-flag composite scoring model, validated via per-flag lift diagnostics before finalisation; 4-tier risk classification
5. **Stratified Sampling** — 5-stratum design (Medium tier split after detecting 2× fraud rate difference between score=1 and score=2); documented methodology refinement
6. **Power BI Export** — 8-sheet Excel export with clean column types, validated 0-null, ready for direct Power BI import

---

## Key Results

| Metric | Value |
|---|---|
| Population | 284,807 transactions |
| Fraud rate (baseline) | 0.1727% |
| Critical tier fraud rate | 1.64% (9.5× baseline) |
| High tier fraud rate | 0.59% (3.4× baseline) |
| Benford MAD (full population) | 0.0211 — Non-Conforming |
| Benford MAD (fraud subgroup) | 0.0529 — 2.5× more deviant |
| Risk score AUC | 0.634 |
| Sample size | 20,109 (7.1% of population) |
| Fraud captured in sample | 56 / 492 (11.4% detection rate) |
| Zero-amount lift | 8.6× baseline |
| Near-duplicate value at risk | $1,070,079 across 69,022 records |

---

## Methodology Notes

**Flag exclusions (validated before scoring):**
- `flag_round_number` excluded — 0 fraud in 1,356 records, 0.00× lift. Round amounts ($100/$500/$1k) are legitimate transaction patterns.
- `flag_benford_digit` narrowed from digits [1, 7, 9] to [7, 9] — digit 1 fired on 48.5% of the population due to $1.00 cluster contamination and sub-$10 transaction density.

**Benford $1.00 sensitivity:**
Original Phase 3 attributed legitimate non-conformity (MAD=0.0211) to the $1.00 cluster. Sensitivity test showed $1.00 removal reduced legitimate MAD 0.0211→0.0160 (24% reduction) but population remained non-conforming. Revised conclusion: $1.00 is a contributing factor, not the primary driver. Residual deviation is driven by broader sub-$10 transaction concentration (83,490 records, 30.8% of legitimate population excluding $1.00).

**Sampling refinement:**
Initial Medium-tier flat sampling rate was revised after detecting that score=1 (fraud rate 0.195%) and score=2 (fraud rate 0.303%) differ by ~2×. Split into Medium_1 and Medium_2 strata with differentiated sampling rates. Result: 31% improvement in fraud capture for 13% additional sample size.

---

## Tech Stack

```
Python          — pandas, numpy, scikit-learn, statsmodels, matplotlib, openpyxl
Power BI        — DAX measures, 8-table data model, 3-page dashboard
Dataset         — Kaggle: Credit Card Fraud Detection (ULB Machine Learning Group)
```

---

## Repository Structure

```
audit-analytics/
├── audit_analytics.ipynb       # Full 6-phase pipeline + Phase 4B validation
├── outputs/
│   ├── exports/
│   │   └── powerbi_export.xlsx # 8-sheet Power BI data model
│   └── charts/                 # Phase output charts (PNG)
├── dashboard/
│   └── audit_dashboard_theme.json
└── README.md
```

---

## Dataset

[Credit Card Fraud Detection](https://www.kaggle.com/datasets/mlg-ulb/creditcardfraud) — ULB Machine Learning Group via Kaggle.

284,807 transactions, 492 fraudulent (0.172%). Features V1–V28 are PCA-transformed for confidentiality. Raw CSV not included in this repo (Kaggle license). Download and place as `creditcard.csv` in the root directory before running the notebook.

---

## How to Run

```bash
pip install pandas numpy scikit-learn statsmodels matplotlib openpyxl
jupyter notebook audit_analytics.ipynb
```

Run cells in order. Phase 4B validation cells (after Phase 4) must run before Phase 6 export — they update the composite score which the export depends on.

Output file written to `outputs/exports/powerbi_export.xlsx`.
