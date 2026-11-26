![RIpart logo]\(library-file\:file_00000000f584820eb212d81d51b59b4a)

Reference Interval Partitioning (RIpart)

Version 4.2.10 · Developed by Alan Carvalho Dias · Powered by LabR Group

RIpart is an R Shiny application for proposing age and sex partitions when establishing laboratory reference intervals. It combines four statistical approaches with repeated subsampling, a standardized difference criterion and a minimum group size. The resulting proposal identifies the age and group combinations for subsequent reference interval estimation and validation.

The workflow accepts numerical laboratory analytes with user-selected age and result columns. A two-level sex or group variable can be included.

# 1. Main features

- Four complementary methods for detecting distributional changes.

- Repeated subsampling to assess support for candidate age cuts and group separation.

- Separate age analyses within groups when separation is retained.

- A final partition table and an `rpart.plot` decision tree built from the retained rules.

- Scatterplots with empirical P50, P10 and P90 profiles.

- A standalone HTML report, numbered CSV tables and automatic PNG export at 600 DPI.

- English and Portuguese interface and report text. Text inside figures remains in English.

- A responsive header and a transparent logo embedded in `app.R` as Base64.

- A configured upper limit of 30 final age-by-group cells.

# 2. Requirements and installation

Install R 4.1 or later. RStudio can be used to run the application. Package installation requires internet access and permission to write to an R package library.

```r
install.packages(c(
  "shiny", "shinythemes", "kableExtra", "readxl",
  "rpart", "rpart.plot", "base64enc"
))
```

The startup code checks these packages and attempts to install any that are missing. Choose an application folder with write access so that results can be saved.

# 3. Run the application

Place `app.R` in the application folder. From that folder, run the following command or select **Run App** in RStudio.

```r
shiny::runApp()
```

An explicit application folder can also be supplied.

```r
shiny::runApp("path/to/RIpart")
```

Use the header links to select English or Portuguese. The URL query parameters `?lang=en` and `?lang=pt` also select the interface language. The logo and demonstration datasets are embedded in the script.

# 4. Input data

Import a `.csv`, `.txt`, `.xls` or `.xlsx` file with a header row and one laboratory result per row. Map the columns in the Configuration tab.

| Column | Requirement | Meaning |
| - | - | - |
| Age | Required | Numerical age in the selected unit |
| Result | Required | Numerical result for the analyte being evaluated |
| Sex or group | Optional | Two levels for the group comparison |

```csv
Age,Sex,Result
24,F,4.65
31,M,5.12
46,F,4.48
58,M,4.91
```

These four rows illustrate the file structure. An analysis requires at least three times the configured minimum group size in valid records before optional outlier removal. With the default minimum of 120, the initial requirement is 360 valid records.

Select years, months or days according to the age values supplied. Selecting a unit labels and interprets those values. Proposed cut points fall on the integer age grid used by the analysis.

For sex analysis, map the female and male codes in the configuration. For another binary grouping variable, use its two levels. With a single selected level, RIpart evaluates age within that level.

Use a dataset prepared for the intended reference population. Rows with missing required values or unparseable numerical values are excluded. Strings such as `<5` are treated as missing numerical results.

# 5. Configure and run an analysis

1. Upload a file or load an embedded demonstration dataset.

2. Select the result, age and optional sex or group columns.

3. Confirm the age unit and the levels to include.

4. Select the detection methods and analysis parameters.

5. Select **Run analysis**.

6. Review Table 1, the final partition and the detailed evidence in the age and group tabs.

The embedded examples comprise a simulated dataset with 8,000 results and the `livertests` dataset with 612 records, age, sex and eight liver analytes.

# 6. Default parameters

| Setting | Default | Effect |
| - | - | - |
| Active methods | All four | Methods contributing to the consensus |
| Minimum results per proposed group | 120 | Sample size criterion for retaining a division |
| Standardized difference tolerance | 0.375 | Minimum difference required by the relevance filter |
| Subsampling repetitions | 100 | Repetitions used to measure support |
| Result transformation | Automatic | Uses natural logarithms for positive data when absolute Bowley skewness decreases by at least 0.05 |
| Outlier removal | Tukey 3 × IQR | Excludes values outside the pooled fences on the analysis scale |
| Method support threshold | 60% | Required subsampling frequency for a method to support a candidate |
| Final group cap | 30 | Configured upper limit for the combined age-by-group proposal |

The interface also offers forced logarithmic transformation, the original scale, Tukey 6 × IQR and an option to retain extremes. Forced logarithmic transformation requires positive results.

The 0.375 tolerance is an exploratory setting. Evaluate the practical and clinical effect of proposed partitions on the reference limits.

# 7. Statistical methods

| Method | Implementation | Distributional feature |
| - | - | - |
| Regression tree | `rpart`, with `cp = 0.01` | Changes in the mean |
| Robust profile segmentation | Dynamic programming implemented in `app.R` | Changes in the age profiles of P50, P10 and P90 |
| Change in mean and variance | Binary segmentation with Gaussian likelihood and a BIC-type penalty | Changes in location and dispersion |
| Change in the tails | Binary segmentation of tail-category counts with a BIC-type penalty | Changes in proportions below pooled P10 and above pooled P90 |

For robust profile segmentation, dynamic programming finds a best-fitting profile segmentation for each evaluated number of cuts. RIpart selects the solution with the largest number of cuts whose adjacent groups meet the tolerance criterion.

For tail segmentation, values equal to the pooled P10 or P90 remain in the central category. The mean-and-variance method uses clipped standardized values in its likelihood calculations.

# 8. Consensus and final partition

The pooled age analysis proposes candidate cut points. Each active method is rerun on random subsamples drawn without replacement. Each subsample contains up to half the results, capped at 50,000 records.

For age candidates, support is evaluated within a window around the proposed position. The window is approximately 3% of the age-grid length, with a minimum of one bin.

A method supports a candidate when its frequency reaches 60%. With four active methods, at least two must support the candidate. With three methods, at least two are required. With two methods, both are required. With one active method, that method supplies the support criterion.

A retained division must also meet the minimum sample size and standardized difference criteria. The script merges adjacent bands when these conditions fail.

Within each pooled age band, the group analysis assesses whether the two group levels should be separated. Consecutive bands with retained group separation form regions in which age is reassessed within each group. The final table combines the retained shared and group-specific boundaries. If the group cap is exceeded, group-specific cuts with the smallest effects are removed first.

# 9. Standardized difference

RIpart uses the largest absolute difference among P50, P10 and P90, divided by a pooled robust standard deviation.

```text
s_pool = sqrt((n_A * s_A^2 + n_B * s_B^2) / (n_A + n_B))

D = max(
  abs(P50_A - P50_B),
  abs(P10_A - P10_B),
  abs(P90_A - P90_B)
) / s_pool
```

Here, `s_A` and `s_B` are the group MAD values on the standard deviation scale. Computation follows the selected analysis scale, including natural logarithms when applied. The relevance filter compares `D` with the configured tolerance.

When the pooled dispersion is zero or undefined, the standardized difference is reported as non-estimable.

# 10. Interpret the results

| Classification | Meaning |
| - | - |
| Confirmed and relevant | Sufficient method support, adequate group sizes and a difference meeting the tolerance |
| Confirmed, not relevant | Sufficient support with a difference below the tolerance |
| Indicative | Some method support, below the required consensus |
| Weak signal | At least two methods reach 30% to below 60% support, outside windows represented by stronger candidates |
| Insufficient n | At least one group fails the minimum size criterion |
| Non-estimable difference | The dispersion needed to standardize the comparison is zero or undefined |

Table 1 combines run details and the overall partition outcome. Table 2 gives the final proposed age-by-group cells. Subsequent tables explain candidate cuts, group comparisons, method support, calculation details and settings. Numbering follows the outputs available in the current run.

Gradual-trend messages identify patterns for which continuous age-specific reference limits may deserve evaluation. Review those messages together with the empirical profiles and the proposed boundaries.

# 11. Tables and figures

The final partition table includes sample size, median, P10, P90 and robust dispersion. These percentiles describe the results within each proposed cell. Estimate and validate the intended reference interval separately for each retained cell.

The decision tree is rendered with `rpart.plot` from the final retained partition. Its nodes display means, counts and percentages.

Scatterplots show a sample of the observations and empirical percentile profiles computed in consecutive age blocks. P50 uses a solid line. P10 and P90 use dashed lines.

Stability plots show how often each method supports age cuts or group separation. Group-effect plots compare standardized differences across age bands and appear when at least two bands have estimable differences.

The group-effect table and plot use descriptive 90% bootstrap intervals based on 200 within-group resamples, preserving the observed group sizes. The candidate-cut position percentiles summarize detected cut locations across subsamples.

All HTML tables use `kableExtra::kbl()`. Text within figures is in English in both interface languages.

# 12. Saved outputs

After each successful analysis, RIpart writes the settings, report, tables, data used and figures under the application working directory.

| Location | Contents |
| - | - |
| `Outputs/saved_params.Rds` | Saved input selections and analysis settings |
| `Outputs/Report_HTML/` | Standalone HTML report with embedded figures and logo |
| `Outputs/Spreadsheets/` | Numbered CSV tables and the records used in the analysis |
| `Outputs/Figures/` | Numbered PNG figures at 600 DPI |

Select **Save everything** to export the current analysis again. Before an analysis, this command saves the settings. Individual figure download buttons have been removed because the figures are exported automatically.

CSV formatting follows the interface language. English uses comma-separated columns and a decimal point. Portuguese uses semicolon-separated columns and a decimal comma.

Preserve a copy of the output folder for each run that needs to be retained. Repeating an analysis with the same analyte name and output numbering can overwrite matching files.

# 13. Troubleshooting

| Problem | Action |
| - | - |
| A required package cannot be loaded | Install the listed packages and check access to the R package library |
| Too few valid records | Check missing values, column mapping, group restrictions and the minimum group size |
| Logarithmic transformation fails | Check for zero or negative results and choose a compatible analysis scale |
| Sex comparison is unavailable | Confirm that the included group variable has exactly two levels and that all levels are selected |
| Output saving fails | Run the app from a folder with write permission and check available disk space |
| An expected partition is absent | Review method support, minimum sample sizes, standardized differences and the group-cap removal column |
| Runtime is long | Review dataset size, active methods and the number of subsampling repetitions |

For reproducibility, retain the input dataset, `app.R` version, saved settings and exported results.

# 14. Authorship and license

Developed by Alan Carvalho Dias, powered by LabR Group.

GPL-3.0, as declared in the application footer.

