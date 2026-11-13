# Data Center Cooling Analysis

**Parent repository:** `EL_5th_Sem`  
**Project status:** Dataset assessment and exploratory analysis setup  
**Working title:** Data Center Cooling — Energy Efficiency, Thermal Performance and Optimization

## Objective

Assess data-center cooling performance using publicly available facility-level, experimental/CFD, and operational datasets. The analysis should distinguish energy efficiency from thermal safety: reducing cooling power is not useful if server inlet or hot-aisle temperatures exceed acceptable limits.

## Research questions

1. What facility and IT power characteristics can be observed across hyperscale / AI data centers?
2. How do airflow, temperature distribution, and cooling-system retrofits affect thermal performance?
3. Which operating variables are useful for predicting hot-aisle / hot-corridor temperature?
4. What evidence supports improvements in cooling efficiency, and what limitations prevent direct comparison across datasets?

## Dataset register

| ID | Source | What it contains | Intended use | Important limitation |
|---|---|---|---|---|
| D1 | [Epoch AI — AI data centers](https://epoch.ai/data/ai-data-centers) | Facility-level estimates of site, IT power, compute and cost, plus separate chiller and cooling-tower CSV tables. The page was updated Nov. 6, 2026 and reported 96 sites, 15.1 GW IT power and 45% coverage. | Macro-level context from facility data; inspect the separate chiller and cooling-tower tables for cooling infrastructure attributes. | The main facility table is not thermal telemetry. Chiller/tower equipment records are not automatically measured cooling energy or temperature time series. Preserve estimates/measurements distinction. Record download date and methodology/version. |
| D2 | [Zenodo — Numerical and experimental dataset for an air-cooled data center](https://zenodo.org/records/7035829) | Experimental temperature data plus OpenFOAM scenarios and post-processed CFD results for previous and retrofitted air-cooled layouts. | Primary engineering dataset for thermal distribution, airflow/CFD comparisons and retrofit analysis. | Large archive (~310.9 MB total); inspect archive structure and metadata before selecting files. The record indicates a newer version exists, so check it before analysis. |
| D3 | [Kula et al. (2025), Energies 18(10), 2581](https://www.mdpi.com/1996-1073/18/10/2581) | Research paper describing operational sensor data and time-series forecasting of warm/hot-corridor temperature. | Methodology/reference paper: preprocessing, 15-minute resampling, feature engineering, temporal validation and model comparison. | This URL is a paper, not a downloadable dataset. Its data availability statement says supporting data are available from the corresponding author on reasonable request. The paper also points to a separate Kaggle dataset by mbjunior; do not confuse that with D4. |
| D4 | [Kaggle — Hyperscale Data Center Dataset](https://www.kaggle.com/datasets/deeplumiere/hyperscale-data-center-dataset) | Dataset listing; exact schema and provenance have not yet been verified. | Candidate for facility-level comparison after inspecting files, columns, units, coverage and provenance. | Do not assume it contains cooling telemetry. Download and inspect the data card, license, files, units and missingness before using it. |

### Source distinction

Epoch AI's page explicitly lists separate downloadable CSVs for data centers, chillers and cooling towers. Use these cooling-equipment tables: they make D1 more relevant to this topic than a site-power-only dataset. However, equipment presence/capacity is not equivalent to measured cooling energy, coefficient of performance (COP), or temperature telemetry. Do not calculate those without the required measurements.


These sources are complementary, not interchangeable:

- **D1:** facility-scale context plus distinct cooling-equipment inventory tables (chillers and cooling towers); verify their columns before claiming efficiency metrics.
- **D4:** candidate facility-scale dataset; verify schema and provenance before assigning a role.
- **D2:** physical thermal/airflow evidence from experiment and CFD.
- **D3:** peer-reviewed modelling methodology and a reference for forecasting thermal conditions; the paper itself does not provide an open raw dataset at this URL.

Do not concatenate these datasets into one table. They describe different entities, spatial scales, time resolutions and measurement processes.

## Proposed repository layout

```text
datacenter cooling/
├── README.md
├── data/
│   ├── raw/              # Original downloads; never edit in place
│   ├── interim/          # Extracted / converted intermediate files
│   └── processed/        # Cleaned analysis-ready tables
├── notebooks/
│   ├── 01_dataset_audit.ipynb
│   ├── 02_epoch_facility_analysis.ipynb
│   ├── 03_zenodo_thermal_analysis.ipynb
│   ├── 04_hyperscale_analysis.ipynb
│   └── 05_cross_source_synthesis.ipynb
├── src/                  # Reusable loading, cleaning and plotting code
├── results/
│   ├── figures/
│   └── tables/
└── reports/              # Final report and presentation assets
```

Create these subdirectories as the work progresses. Keep large raw archives out of Git unless repository size and source licensing make that appropriate; prefer source links, checksums, and reproducible download instructions.

## Analysis plan

### Phase 1 — Dataset audit (do this first)

For every downloaded file, record:

- Source URL, access date, version and license.
- Filename, file format, size and checksum.
- Number of rows/columns (where applicable), time span and sampling interval.
- Variable name, physical meaning, unit, sensor/location and data type.
- Missing values, duplicates, invalid values and outliers.
- Whether values are measured, simulated, estimated or derived.

Produce a dataset inventory table and a data dictionary before modelling.

### Phase 2 — Exploratory analysis

**Facility-scale (D1 and verified D4):**
- Site count and geographic distribution.
- IT power and compute-capacity distributions.
- Facility power estimates only where the source explicitly provides them.
- Compare scale and capacity; do not infer site-level cooling efficiency from IT power alone.

**Thermal / CFD (D2):**
- Compare baseline and retrofitted layouts under equivalent operating conditions.
- Plot temperature distributions and spatial hot spots.
- Compare temperature at relevant inlet/outlet or cold/hot aisle locations where the metadata supports it.
- Inspect airflow/velocity fields and pressure/flow metrics if present.
- Quantify changes using consistent regions, boundary conditions and units.

### Phase 3 — Modelling (only if the data supports it)

A defensible first model is hot-aisle / hot-corridor temperature forecasting using time-ordered sensor data.

- Baseline: persistence / last-value forecast and linear regression.
- Models: Random Forest and XGBoost; consider a time-series model only after establishing a baseline.
- Split chronologically; never randomly split time-series observations.
- Fit preprocessing only on training data.
- Report MAE, RMSE and normalized RMSE, plus residual plots.
- Inspect errors at high temperatures separately. Underpredicting thermal extremes is a safety-relevant failure.
- Avoid claiming cooling-energy optimization unless the dataset contains suitable cooling power and operational variables.

### Phase 4 — Synthesis and engineering conclusions

Separate findings into:
1. What the data directly demonstrates.
2. What is inferred, with assumptions stated.
3. What cannot be concluded because measurements are absent or datasets are incompatible.

## Minimum deliverables for a presentable submission

- [ ] Dataset register with provenance, license and access date.
- [ ] Data dictionary with units and sensor/variable definitions.
- [ ] Reproducible data-loading and cleaning scripts.
- [ ] Dataset quality report (missingness, duplicates, ranges, outliers).
- [ ] At least 5 clearly labelled, unit-aware figures.
- [ ] Baseline-versus-retrofit thermal comparison from D2.
- [ ] Facility-scale context analysis from D1 (and D4 only after verification).
- [ ] A justified model evaluation if time-series data is usable.
- [ ] Final report: objective, sources, methodology, results, limitations, conclusion and references.
- [ ] README instructions for reproducing the analysis.

## Presentation checklist

- State the research question and scope on the first page.
- Label every axis with variable and unit.
- Include sample count, time period and source below each figure.
- Use consistent terminology: IT power, facility power, cooling power, temperature, airflow.
- Explain every acronym at first use (e.g. PUE, CFD, HVAC).
- Cite figures and datasets; state licenses and modifications.
- Never present estimates as measurements or correlation as causation.
- Keep raw data immutable and make every cleaning step reproducible.

## Important metrics and equations

Power Usage Effectiveness:

`PUE = Total Facility Energy / IT Equipment Energy`

A lower PUE indicates lower facility overhead relative to IT energy, but it is not a direct measure of cooling efficiency alone. It is affected by non-IT loads and operating conditions.

For a measured baseline-to-retrofit comparison:

`Percentage change = ((Retrofit value - Baseline value) / Baseline value) × 100`

State whether a negative change is desirable for the particular metric. For temperature, compare both average and maximum / high-percentile values; averages alone can hide hot spots.

## Immediate next steps

1. Download D2's experimental temperature archive first; inspect its README/metadata and file tree before touching the full CFD archives.
2. Download D1's main AI data-center CSV and its separate `Data center cooling: Chillers` and `Data center cooling: Cooling towers` CSVs. The page documents all three downloads; retain them as separate tables and preserve the access date/version.
3. Obtain D4 and inspect its data card, schema and license. Do not use it until its contents are verified.
4. Treat D3 as a methodology source; if its underlying operational data is required, follow the paper's data-request route or evaluate the separate dataset linked by the paper.
5. Build the dataset audit notebook before producing charts or training models.

## References

- Epoch AI, AI data centers: https://epoch.ai/data/ai-data-centers
- Kuzay et al., Numerical and experimental dataset for an air-cooled data center, Zenodo, DOI: 10.5281/zenodo.7035829
- Kula et al. (2025), Modelling the Temperature of a Data Centre Cooling System Using Machine Learning Methods, Energies 18(10), 2581. https://doi.org/10.3390/en18102581
- Kaggle, Hyperscale Data Center Dataset: https://www.kaggle.com/datasets/deeplumiere/hyperscale-data-center-dataset
