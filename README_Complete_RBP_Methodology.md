# Complete RBP Methodology Notebook

**Full Replication Guide: From Raw Data to Exhibits**

This notebook provides a complete, step-by-step replication of the Relevance-Based Prediction (RBP) methodology, from initial data loading through all processing steps to final exhibits and hypothesis tests.

---

## 📋 Overview

The `Complete_RBP_Methodology.ipynb` notebook demonstrates the entire RBP pipeline used for forecasting 63-day realized volatility. It is designed to be:

- **Fully Replicable**: Shows every step from raw data to final results
- **Educational**: Explains each methodology component with equations and code
- **Publication-Ready**: Clean, well-documented code suitable for academic use
- **Comprehensive**: Includes all 5 exhibits with visualizations and findings

**Reference Paper:** "Relevance-Based Prediction" (SSRN-5631270)  
**Target Variable:** 63-day realized volatility  
**Sample Period:** 355 prediction dates (monthly, 1996-2024)

---

## 🎯 What This Notebook Contains

### Part 1: Data Loading and Preparation
- Overview of data requirements
- Panel data structure
- Target and predictor variables

### Part 2: Core Methodology - Relevance and Informativeness
- **Similarity** (Equation 2): Mahalanobis distance between observations
- **Informativeness** (Equations 3-4): Unusualness of observations
- **Relevance** (Equation 1): Combined similarity and informativeness
- Full implementation with covariance shrinkage and PSD handling

### Part 3: Partial Weights (Equations 5-8)
- Censoring/retention mechanism
- Weight computation with bias correction
- Lambda optimization (simplified version)

### Part 4: Grid Search and Adjusted Fit
- Grid cell evaluation (variable subsets × retention fractions)
- Cell prediction computation
- **Adjusted Fit** (Equations 20-23): Cell reliability metric

### Part 5: Composite Prediction (Equations 24-25)
- Cell weighting by adjusted fit
- Composite prediction aggregation
- Composite fit calculation

### Part 6: Solo Distributions (Equations 13-14, 18)
- **Solo Predictions** (Equation 14): Individual observation predictions
- **Contribution Weights** (Equation 18): Individual contribution to composite
- Distribution metrics (IQR, Range 5-95, weighted mean)

### Part 7: Processing Results for Exhibits
- Error metrics (scaled by realized volatility)
- Classifications (IQR-only, 5-95 Range-only)
- Volatility regimes (LOW, MID, HIGH)
- Fit quintiles

### Part 8: Exhibits and Findings

#### **Exhibit A: Forecast Error by Fit Quintile**
- **Research Question:** Does ex-ante composite reliability predict forecast accuracy?
- **Key Finding:** Correlation increases 4.6× from Q1 (0.159) to Q5 (0.738)
- **Visualization:** Scaled error and correlation by quintile

#### **Exhibit B: Solo Distribution Width vs Forecast Error**
- **Research Question:** Does dispersion of solo predictions relate to forecast error?
- **Key Finding:** TIGHT distributions perform best (~24.75% error), MODERATE worst (~30% error)
- **Visualization:** Error by width classification (IQR-only and 5-95 Range-only), scatter plot

#### **Exhibit C: Fit Across Volatility Regimes**
- **Research Question:** How do confidence signals behave across volatility environments?
- **Key Finding:** MID volatility shows best accuracy (21.38% error) despite lowest fit
- **Visualization:** Fit, IQR, and error by volatility regime
- **Note:** Descriptive only (regimes are defined splits)

#### **Exhibit D: High Volatility vs Wide Distribution Overlap**
- **Research Question:** Within HIGH volatility periods, does distribution width matter?
- **Key Finding:** HIGH+WIDE shows BEST performance (22.63% error) - counter-intuitive result
- **Visualization:** Error and fit by width within HIGH volatility

#### **Exhibit E: Fit by Distribution Width Classification**
- **Research Question:** What are the fit values for each distribution width?
- **Key Finding:** TIGHT shows highest mean fit (~352-370), WIDE has extreme outliers
- **Visualization:** Mean and median fit by classification

---

## 🚀 How to Use This Notebook

### Prerequisites

1. **Python Environment**
   - Python 3.8+
   - Required packages:
     - `pandas`
     - `numpy`
     - `scipy`
     - `matplotlib`
     - `seaborn`
     - `pathlib`

2. **Data Files**
   - The notebook loads pre-processed exhibit CSV files from:
     - `D:\RPB1_Outputs\new_exhibits\`
   - Required CSV files:
     - `exhibit_a_fit_quintiles.csv`
     - `exhibit_b_width_regimes_iqr_only.csv`
     - `exhibit_b_width_regimes_range_5_95_only.csv`
     - `exhibit_c_volatility_regimes.csv`
     - `exhibit_d_high_vol_by_width_iqr_only.csv`
     - `exhibit_d_high_vol_by_width_range_5_95_only.csv`
     - `exhibit_e_fit_by_width_iqr_only.csv`
     - `exhibit_e_fit_by_width_range_5_95_only.csv`
     - `exhibit_b_scatter_data_iqr_only.csv`

### Running the Notebook

1. **Open the Notebook**
   ```bash
   jupyter notebook Complete_RBP_Methodology.ipynb
   ```
   Or use JupyterLab, VS Code, or any Jupyter-compatible environment.

2. **Execute Cells Sequentially**
   - The notebook is designed to run from top to bottom
   - Each part builds on previous parts
   - Methodology sections define functions used in later sections

3. **View Results**
   - Tables display automatically using `display()`
   - Visualizations appear inline
   - Key findings are summarized in markdown cells

### For Full Replication

To replicate from raw data (not just load pre-processed CSVs):

1. **Run the Full Pipeline**
   - Execute `RBP_Volatility_Forecasting_Pipeline.py` first
   - This generates all the CSV files used in Part 8

2. **Modify Part 1**
   - Replace the data loading placeholder with actual data loading code
   - Load daily SPX returns, compute 63-day realized volatility
   - Load monthly macro variables from FRED
   - Merge datasets

3. **Run Parts 2-6**
   - These contain the core methodology code
   - Can be run independently for each prediction date
   - Functions are self-contained and well-documented

---

## 📊 Key Methodology Components

### 1. Relevance Computation (Equations 1-4)

**Similarity:**
$$\text{sim}(x_i, x_t) = -\frac{1}{2} (x_i - x_t)^T \Omega^{-1} (x_i - x_t)$$

**Informativeness:**
$$\text{info}(x, \bar{x}) = (x - \bar{x})^T \Omega^{-1} (x - \bar{x})$$

**Relevance:**
$$r_{it} = \text{sim}(x_i, x_t) + \frac{1}{2} [\text{info}(x_i, \bar{x}) + \text{info}(x_t, \bar{x})]$$

**Critical:** Relevance is centered to zero mean before computing weights.

### 2. Partial Weights (Equations 5-8)

Weights are computed with censoring (retaining only top observations by relevance):

$$w_{it,\theta} = \frac{\exp(\lambda r_{it}) \delta(r_{it})}{\sum_j \exp(\lambda r_{jt}) \delta(r_{jt})}$$

Where $\delta(r_{it})$ is the censoring indicator (1 if retained, 0 if censored).

### 3. Adjusted Fit (Equations 20-23)

Cell reliability metric:

$$\text{adjusted\_fit} = K \times (\text{fit} + \text{asymmetry})$$

Where:
- $K$ = number of variables in subset
- $\text{fit} = \rho(w, y)^2$ (squared correlation)
- $\text{asymmetry}$ = penalty for asymmetric weight distribution

### 4. Composite Prediction (Equations 24-25)

Cell weights:
$$w_{\text{cell},j} = \frac{\text{adjusted\_fit}_j}{\sum_k \text{adjusted\_fit}_k}$$

Composite prediction:
$$\hat{y}_{\text{composite}} = \sum_j w_{\text{cell},j} \times \hat{y}_{\text{cell},j}$$

### 5. Solo Distributions (Equations 13-14, 18)

Solo prediction:
$$\text{solo}(i) = \bar{y} + \frac{\text{info}(x_t)}{r_{it}} \times (y_i - \bar{y})$$

Contribution weight:
$$\xi_i = \frac{r_{it}^2}{\sum_j r_{jt}^2}$$

---

## 🔬 Statistical Tests

The notebook references statistical tests performed on the exhibits:

- **Exhibit A:** Kruskal-Wallis + pairwise Mann-Whitney U (fit quintiles)
- **Exhibit B:** Kruskal-Wallis + pairwise Mann-Whitney U (distribution width)
- **Exhibit D:** Kruskal-Wallis + pairwise Mann-Whitney U (HIGH vol by width)
- **Exhibit E:** Kruskal-Wallis + pairwise Mann-Whitney U (fit by width)
- **Exhibit C:** No tests (descriptive only)

**Test Method:** Non-parametric (Kruskal-Wallis, Mann-Whitney U) due to:
- Potential temporal dependence (time-ordered dates)
- Different distributions across groups
- More robust to violations

**Correction:** Bonferroni for multiple comparisons

See `statistical_tests_exhibits.py` for full test implementation.

---

## 📁 File Structure

```
RPB1/
├── Complete_RBP_Methodology.ipynb          # This notebook
├── README_Complete_RBP_Methodology.md       # This README
├── RBP_Volatility_Forecasting_Pipeline.py  # Full pipeline (generates CSVs)
├── create_exhibits_separate_classifications.py  # Exhibit generation
├── statistical_tests_exhibits.py           # Statistical tests
└── D:\RPB1_Outputs\new_exhibits\          # Pre-generated exhibit CSVs
    ├── exhibit_a_fit_quintiles.csv
    ├── exhibit_b_width_regimes_iqr_only.csv
    ├── exhibit_b_width_regimes_range_5_95_only.csv
    ├── exhibit_c_volatility_regimes.csv
    ├── exhibit_d_high_vol_by_width_iqr_only.csv
    ├── exhibit_d_high_vol_by_width_range_5_95_only.csv
    ├── exhibit_e_fit_by_width_iqr_only.csv
    ├── exhibit_e_fit_by_width_range_5_95_only.csv
    └── exhibit_b_scatter_data_iqr_only.csv
```

---

## 🎓 Educational Value

This notebook is designed for:

1. **Researchers** wanting to understand the RBP methodology
2. **Practitioners** implementing RBP for their own applications
3. **Students** learning relevance-based prediction methods
4. **Reviewers** verifying the methodology and results

### Learning Path

1. **Start with Part 1-2**: Understand data structure and relevance computation
2. **Study Part 3-4**: Learn weight computation and grid search
3. **Examine Part 5-6**: Understand composite prediction and solo distributions
4. **Review Part 7**: See how results are processed for analysis
5. **Analyze Part 8**: Explore exhibits and findings

---

## 🔍 Key Insights from Exhibits

### Exhibit A: Fit Predicts Correlation
- **Strong relationship**: Fit quintile explains 4.6× improvement in correlation
- **Moderate relationship**: Fit is less predictive of absolute error
- **Implication**: Fit is a reliable ex-ante signal for prediction quality

### Exhibit B: Distribution Width Matters
- **TIGHT performs best**: Lowest error (~24.75%)
- **MODERATE performs worst**: Highest error (~30%)
- **WIDE is intermediate**: ~27-29% error
- **Weak correlation**: IQR and error correlation is only 0.0644
- **Implication**: Classification matters more than continuous IQR

### Exhibit C: Volatility Regimes
- **MID volatility is optimal**: Best accuracy (21.38%) despite lowest fit
- **LOW volatility is challenging**: Worst accuracy (44.92%)
- **HIGH volatility shows high fit**: 414.51 fit, intermediate accuracy (25.95%)
- **Implication**: Fit and accuracy are not always aligned

### Exhibit D: HIGH+WIDE Surprise
- **Counter-intuitive result**: HIGH+WIDE shows BEST performance (22.63% error)
- **Strong fit signal**: 561.16 fit (highest among HIGH vol periods)
- **Implication**: Wide distributions in crisis periods may capture uncertainty better

### Exhibit E: Fit by Width
- **TIGHT has highest fit**: ~352-370 mean fit
- **WIDE has extreme outliers**: Mean inflated, median low (~168-173)
- **MODERATE has lowest fit**: ~276-283
- **Implication**: Fit distribution is highly right-skewed for WIDE

---

## ⚠️ Important Notes

1. **Simplified Code**: Some functions (e.g., lambda optimization) are simplified for clarity. The full pipeline (`RBP_Volatility_Forecasting_Pipeline.py`) contains complete implementations.

2. **Pre-processed Data**: Part 8 loads pre-generated CSV files. For full replication, run the complete pipeline first.

3. **Computational Requirements**: The full pipeline is computationally intensive (575 grid cells × 355 dates). The notebook focuses on methodology, not full execution.

4. **Classification Methods**: Exhibits B, D, E use **separate** classifications:
   - IQR-only: Based solely on IQR percentiles
   - 5-95 Range-only: Based solely on 5-95 range percentiles
   - Not combined (no "AND" logic)

5. **Volatility Regimes**: Exhibit C uses symmetric splits (25-50-25%) for LOW, MID, HIGH.

---

## 📚 References

- **Paper**: "Relevance-Based Prediction" (SSRN-5631270)
- **Methodology Document**: `D:\RPB1_Outputs\EXHIBIT_METHODOLOGY.md`
- **Findings Summary**: `D:\RPB1_Outputs\EXHIBIT_FINDINGS_SUMMARY.md`

---

## 🤝 Contributing

This notebook is part of a research project. For questions or issues:

1. Check the methodology document for detailed explanations
2. Review the findings summary for interpretation guidance
3. Examine the full pipeline code for implementation details

---

## 📝 License

This notebook is provided for research and educational purposes as part of the RBP methodology replication project.

---

## ✅ Checklist for Replication

- [ ] Install required Python packages
- [ ] Verify data directory path (`D:\RPB1_Outputs\new_exhibits\`)
- [ ] Check that all CSV files exist
- [ ] Run notebook cells sequentially
- [ ] Verify visualizations render correctly
- [ ] Compare results with findings summary
- [ ] (Optional) Run full pipeline for complete replication

---

**Last Updated:** 2024  
**Notebook Version:** 1.0  
**Status:** Complete and Ready for Use

