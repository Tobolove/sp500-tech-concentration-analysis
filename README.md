# How Diversified Is an S&P 500 ETF?

**Technology concentration and revenue growth in the S&P 500 at the end of 2024**

People often call an S&P 500 ETF a broad investment in 500 companies. But the index is weighted by market value, so a few large technology companies get a big share of the money. This project measures how concentrated the index is. It then tests whether technology companies grew their revenue faster than the rest, which would be one reason for their large weight.

Capstone Project 2 · Master's Degree in AI, Woolf University · Tobias Frei · 2026-11-26

> This is a statistics exercise, not investment advice.

## Key findings

| Question | Result |
|---|---|
| How much of the market value does Technology hold? | **32.5%** of the market value, from **16.4%** of the companies |
| How much do the 10 largest companies hold? | **38.9%** of the market value, from **2.0%** of the companies |
| Mean revenue growth, Technology vs. other sectors | 11.6% vs. 6.2%, a difference of 5.4 percentage points |
| Welch t-test (two-sided, α = 0.05) | t = 1.80, df = 91.7, p = 0.076, 95% CI -0.57 to 11.44 → **H0 not rejected** |
| Bootstrap, difference in means (10,000 resamples) | 95% interval -0.02 to 11.76, includes zero |
| Bootstrap, difference in medians | 2.6 points, 95% interval 0.6 to 6.3, excludes zero |

**In short:** technology carries a heavy weight in the index, but the sector as a whole does not have a statistically clear growth lead to justify it. The typical tech company grew a little faster (see the medians). The bigger gap in the means comes from a few outliers: Super Micro Computer (+143%), NVIDIA (+122%) and Micron (+93%). The analysis plan named the test of the means as the main test before the groups were compared, so that test stays the main result.

Eight of the ten largest companies are tech-related. The dataset still places Alphabet and Meta in Communication Services and Amazon and Tesla in Consumer Cyclical. "Technology" in this dataset is therefore a narrower group than what people usually mean by "big tech".

## Repository contents

| File | Content |
|---|---|
| [analysis.ipynb](analysis.ipynb) | The full analysis: setup, data investigation, cleaning, descriptive statistics, four figures, hypothesis test, bootstrap |
| [module_summary.pdf](module_summary.pdf) | The written report: methods, results, interpretation, limitations, references. [module_summary.docx](module_summary.docx) is the editable source. |
| [sp500_companies.csv](sp500_companies.csv) | The dataset, unchanged from Kaggle |
| [requirements.txt](requirements.txt) | Exact package versions of the environment the notebook ran in |

## Dataset

- **Source:** `sp500_companies.csv` from *S&P 500 Stocks (daily updated)* by Larxel, version 1023, on [Kaggle](https://www.kaggle.com/datasets/andrewmvd/sp-500-stocks), CC0 licence.
- **Snapshot date:** 2024-12-20. The file has no date column. This date is the last trading day in the price files of the same version, which was published on 2024-12-23.
- **Origin:** the Kaggle page does not name a source, but the columns and sector labels match Yahoo Finance.
- **Size:** 502 rows (one per listed share) and 16 columns.
- **Columns used:** `Sector`, `Marketcap` and `Revenuegrowth`. Revenue growth compares one quarter with the same quarter a year earlier.

### From raw data to the analysis set

```
502 rows  →  499 companies  →  496 companies with revenue growth
                                (81 Technology, 415 other sectors)
```

1. **Share classes:** Alphabet, Fox and News Corp each appear twice because they have two share classes. Both rows of a company have the same sector and growth, so the analysis keeps one row per company.
2. **Missing outcome:** Verizon, American Tower and Western Digital have no revenue growth value. They are left out of the test (0.6% of the companies).
3. **Unused columns:** `Ebitda` is missing for 29 companies, all of them in Financial Services, where banks and insurers do not report it. `Fulltimeemployees` has 9 gaps. Neither column is needed for the research question.

## Methods

- **Analysis plan first.** The hypotheses, α, the test and the robustness check were fixed before the groups were compared. This follows the initial data analysis approach of Lusa et al. (2024).
- **Data investigation:** missing values, duplicates, plausibility of the outcome (range, zeros used as placeholders, extreme values checked against quarterly reports) and category sizes.
- **Descriptive statistics:** market value (skewness 8.16) and revenue growth (skewness 3.82) are both right-skewed, so the notebook reports medians and quartiles alongside the means.
- **Figures:**
  - a histogram of revenue growth
  - a box plot of revenue growth by group
  - a bar chart of each sector's share of the market value next to its share of the companies
  - a histogram of the bootstrap differences
- **Test:** Welch's two-sample t-test, `scipy.stats.ttest_ind(tech, other, equal_var=False)`, with
  - H0: μ_Tech = μ_Other
  - H1: μ_Tech ≠ μ_Other

  The technology group is five times smaller than the other group, and its standard deviation is 1.66 times larger. Student's t-test would therefore underestimate the uncertainty of the tech mean, so Welch's version is the right default here (Delacre et al., 2017).
- **Robustness:** 10,000 bootstrap resamples of the differences in means and in medians. The bootstrap distribution of the mean difference is almost symmetric (skewness 0.25), which supports the test's assumption of a roughly normal sampling distribution.

## Reproducing the analysis

You need Python 3.12. The notebook ran on 3.12.7.

```bash
git clone https://github.com/Tobolove/sp500-tech-concentration-analysis.git
cd sp500-tech-concentration-analysis
python -m venv .venv
# Windows: .venv\Scripts\activate      macOS/Linux: source .venv/bin/activate
pip install -r requirements.txt
jupyter lab analysis.ipynb
```

In JupyterLab, choose **Kernel → Restart Kernel and Run All Cells**. Or run the notebook without opening it:

```bash
jupyter nbconvert --to notebook --execute analysis.ipynb --output analysis_executed.ipynb
```

The setup cell sets the random seed (`SEED = 42`) once, and the bootstrap draws from NumPy's global random generator. The bootstrap numbers therefore only match when all cells run in order, top to bottom, in a single session. A clean run like that reproduces every number in this README.

If you use uv instead of pip, create the environment with `uv venv --python 3.12` and install the packages with `uv pip install -r requirements.txt`.

## Limitations

- **A single snapshot.** The data describe the end of 2024 only. One quarter of revenue growth says little about long-term growth, and the latest quarter ends at different dates for different companies.
- **No return data.** The daily price file of the same Kaggle dataset is about two-thirds empty, so stock returns could not be analysed.
- **Selection.** The index contains only large, mostly US companies that were members at the time of the snapshot. The results do not carry over to smaller companies or other years.
- **Sector labels.** If Alphabet, Meta, Amazon and Tesla were counted as Technology, the result could change.
- **Weights.** The analysis uses market value, not the official float-adjusted index weight.
- **Independence.** Companies in the same sector and market are only partly independent of each other. All results describe associations, not causes.
- **One company missing.** The file holds 499 companies instead of 500, and it is still unclear which constituent is missing.

## Possible next steps

- Find out which company is missing from the file.
- Compare several snapshots over time.
- Analyse revenue growth over several quarters instead of one.
- Repeat the test with Alphabet, Meta, Amazon and Tesla counted as Technology.

## References

Delacre, M., Lakens, D., & Leys, C. (2017). Why psychologists should by default use Welch's t-test instead of Student's t-test. *International Review of Social Psychology, 30*(1), 92-101. https://doi.org/10.5334/irsp.82

Larxel. (2024). *S&P 500 stocks (daily updated)* (Version 1023) [Data set]. Kaggle. https://www.kaggle.com/datasets/andrewmvd/sp-500-stocks

Lusa, L., Proust-Lima, C., Schmidt, C. O., Lee, K. J., le Cessie, S., Baillie, M., Lawrence, F., & Huebner, M. (2024). Initial data analysis for longitudinal studies to build a solid foundation for reproducible analysis. *PLOS ONE, 19*(5), Article e0295726. https://doi.org/10.1371/journal.pone.0295726
