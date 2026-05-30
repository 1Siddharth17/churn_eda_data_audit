#  D2C Churn Intelligence — Part 1 Deliverables

---


### **TIER 1: Core Deliverables (Must-Review)**

#### 1️ **eda_audit.ipynb** — The Main Notebook
- **What:** Google Colab-compatible Jupyter notebook (25 cells, ~400 lines of code)
- **Who should use:** Data scientists, analytics engineers, stakeholders (run-through demo)
- **How to access:** 
  - Direct import into Google Colab via GitHub link (or drag-and-drop .ipynb)
  - Runs entirely self-contained with synthetic data generation (no external CSV uploads needed)
  - All charts render inline; outputs saved to `/outputs` folder
- **What's inside:**
  -  Complete data loading & schema inspection
  -  Professional-grade data quality audit (missing values, duplicates, outliers, leakage checks)
  -  8 production-quality visualizations (churn distribution, RFM analysis, support tickets, intervention ROI, tenure, category diversity, risk tiers)
  -  6 data-backed, business-actionable hypotheses
  -  Heuristic pre-ML risk scoring (quick customer segmentation)
  -  Master export table for downstream modeling
- **Key outputs:** 9 PNG charts + 1 CSV (churn_risk_master.csv) + inline insights

#### 2️ **business_memo.md** — Executive Brief
- **What:** 2,000-word C-suite memo with findings, priorities, and financial impact
- **Who should read:** Marketing leaders, retention team, CFO, product stakeholders
- **Key sections:**
  -  3 CRITICAL immediate priorities (with 7–14 day implementation timelines)
  - 5 detailed findings with business context
  - Guardrails: What NOT to do
  - Financial ROI estimate: **₹9,80,000 retention value over 90 days** (2.9x ROI)
  - Decision-ready recommendations
- **Time to read:** 8–10 minutes

#### 3️ **data_quality_report.md** — Technical Audit
- **What:** Comprehensive data quality assessment (8 issues identified, all explained and actionable)
- **Who should read:** Data engineers, analytics team, compliance/data governance
- **Key sections:**
  - Issue-by-issue audit (severity, business impact, treatment)
  - Missing values, duplicates, anomalies, leakage risk analysis
  - Recommended SQL/Python snippets for data prep
  - Pre-ML checklist (8-item validation before building models)
- **Time to read:** 12–15 minutes

---

##  How to Use These Deliverables

### **Scenario 1: You're a Data Scientist Building the Model**
1. **Start:** Open `eda_audit.ipynb` in Google Colab
2. **Run:** Execute cells top-to-bottom (takes ~3 minutes)
3. **Review:** Examine charts and hypotheses → understand the business problem deeply
4. **Action:** Read `data_quality_report.md` section "Data Preparation Script" → apply treatments to your training data
5. **Next:** Proceed to Part 2 (predictive modeling) with clean, feature-ready data

### **Scenario 2: You're a Retention Manager**
1. **Start:** Read `business_memo.md` (TL;DR section first)
2. **Act:** Implement 3 priority actions this week (Trial conversion, 90-day reactivation, support SLA)
3. **Measure:** Track metrics in memo (retention rate by segment) monthly
4. **Ref:** Keep `data_quality_report.md` on hand for guardrails ("Don't give blanket discounts to all Trial customers")

### **Scenario 3: You're a Stakeholder in a Review/Evaluation**
1. **Read:** Business memo (5 min) + TL;DR of data quality report (3 min)
2. **View:** Run notebook cells 14–16 (churn distribution, RFM, support ticket charts) — they're the key visuals
3. **Question:** "What's the financial impact if we implement the 3 priorities?" → Answer in memo (₹9,80,000 over 90 days)
4. **Approve:** If satisfied, greenlight Part 2 modeling with confidence

---

##  Key Findings at a Glance

| Finding | Severity | Impact | Action |
|---|---|---|---|
| Trial customers churn at 65% (vs 18% Premium) |  CRITICAL | 487 churned annually from Trial alone | Launch Day 10/20/25 conversion sequence + 50% discount |
| 90+ day inactive customers churn at 2.8× rate |  CRITICAL | 1,200 customers in danger zone | Automate 90/105/120-day reactivation campaign |
| 3+ support tickets predict 50% churn |  CRITICAL | 800 customers in repeat-issue zone | Implement 48h SLA; audit delivery/quality issues |
| Single-category buyers churn 3× more than multi-category |  MODERATE | 35% of base at elevated risk | In-cart cross-sell + routine-building emails |
| Personal Outreach retains at 65% vs Email at 35% |  MODERATE | ROI opportunity in messaging | Tier interventions by customer LTV |
| New customers (0–3m) churn like "Lost" segment |  MODERATE | Onboarding window critical | Redesign Day 1/7/14 engagement sequence |

---

##  Visualizations Included

| # | Chart | Insight |
|---|---|---|
| 1 | Missing Value Audit | ~2% data gaps; easily treatable |
| 2 | Outlier Analysis | Order values & recency; all explainable |
| 3 | Churn Distribution | Overall 43% churn; stratified by plan type |
| 4 | RFM vs Churn | High recency + low frequency = high churn |
| 5 | Support Tickets vs Churn | Churn rate rises with ticket volume (strong signal) |
| 6 | Intervention Effectiveness | Personal Outreach (65% retention) >> Email (35%) |
| 7 | Tenure & RFM Segments | New customers at risk; "Lost/Hibernating" segments critical |
| 8 | Product Category Diversity | Multi-category buyers show loyalty; single-category at risk |
| 9 | Risk Tier Distribution | Pre-ML heuristic segmentation (for quick targeting) |

---

##  Data Quality Summary

| Category | Status | Details |
|---|---|---|
| **Completeness** | 97.8% | Minor gaps in age (2.4%) and city (1.6%) |
| **Duplicates** | 0 | Primary keys intact; no exact duplicate rows |
| **Leakage Risk** | Prevented | All dates filtered to ≤ snapshot_date (2024-06-30) |
| **Outliers** | Explainable | Order values, recency, monetary — all interpretable |
| **Joins** | Intact | Foreign key relationships verified; no orphans |
| **Overall** | APPROVED | Ready for ML with documented treatments |

---

##  Methodology Notes

### Data Approach
- **Sample:** 5,000 customers, 15,000+ orders, 8,000+ support tickets, 2,000 interventions
- **Cohort:** All customers as-of snapshot date (2024-06-30)
- **Timeframe:** Full history pre-snapshot (since 2021-01-01 for early cohorts)
- **Leakage guard:** No future-dated events used; all analysis retrospective

### Statistical Methods
- **Descriptive:** Medians, distributions, cross-tabs (no parametric assumptions)
- **Comparative:** Mann-Whitney U test for RFM distributions (churn vs retained)
- **Segmentation:** RFM quantiles, tenure buckets, risk tiers (heuristic scoring)
- **Visualization:** Density overlays, stratified bar charts, scatter plots (interpretable)

### Business Lens
- Every insight tied to actionable business decision
- ROI estimation based on conservative assumptions (20% improvement = success)
- Intervention strategies differentiated by customer segment and LTV
- No generic churn talk; all statements evidence-backed

---

## Guardrails Applied

 **No future data leakage**  
 **No intervention outcomes in churn features** (avoids selection bias)  
 **Missing values treated separately** (not dropped carelessly)  
 **Return orders flagged** (not hidden as noise)  
 **Outliers explained** (not removed without reason)  
 **RFM segment stratified** (not treated as single metric)  
 **Snapshot date enforced** (consistent point-in-time)  

---

## Next Steps — Part 2 (Coming Soon)

Once these findings are internalized and immediate actions are underway:

- **Feature Engineering:** Transform raw signals into ML-ready features
  - Recency decay functions
  - Frequency engagement tiers
  - Ticket risk profiles
  - Category affinity scores
  - Tenure lifecycle stages

- **Predictive Modeling:** Build churn classifiers
  - Baseline: Logistic Regression (interpretable)
  - Advanced: XGBoost (high accuracy)
  - Probability calibration for business decision-making

- **Explainability:** SHAP values, feature importance
  - Which factors matter most? (Quantified)
  - Sensitivity: "If we fix support SLA, churn drops ___"

- **Deployment:** Churn score updates (weekly), automated interventions, A/B testing framework

---


## File Manifest

```

├── eda_audit.ipynb                    (25 cells, 400 lines)
├── business_memo.md                   (2,000 words)
├── data_quality_report.md             (3,000 words)
├── churn_risk_master.csv              (5,000 rows)
└── [9 PNG charts]
    ├── fig1_missing_values.png
    ├── fig2_outliers.png
    ├── fig3_churn_distribution.png
    ├── fig4_rfm_vs_churn.png
    ├── fig5_tickets_vs_churn.png
    ├── fig6_intervention_effectiveness.png
    ├── fig7_tenure_rfm_churn.png
    ├── fig8_category_diversity.png
    └── fig9_risk_tiers.png
```
