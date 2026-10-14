# Technical Results

[Back to project overview](README.md)

This document records the methodology, notebook outputs, and technical limitations of the beehive analysis. It separates what the notebooks actually demonstrate from what still requires validation.

## Scope of this report

The raw datasets are not available in the repository, so the figures below are transcribed from the outputs stored in the notebooks. They have not been independently reproduced during this review.

Two analyses are included:

| Analysis | Raw records | Daily records | Purpose | Outcome |
| --- | ---: | ---: | --- | --- |
| American dataset | 1,355 | 195 | Estimate image-derived bee activity from environmental and capture features | Preliminary regression results produced |
| Austrian dataset | 15,065 | 392 | Explore environmental time series and estimate daily temperature | Model training fails because of missing values |

Neither notebook uses a labelled colony-health event as its target. The results therefore measure count or temperature estimation, not hive-health prediction.

## American dataset

### Input data

Notebook: [`code/Time-series-analysis_america_db.ipynb`](code/Time-series-analysis_america_db.ipynb)

The source file is `bee_data.csv`. The stored records run from 17 April through 28 October 2023 and are described in the notebook as approximately two-hour observations.

The raw schema contains:

| Field | Interpretation in the notebook |
| --- | --- |
| `datetime` | Observation timestamp |
| `temperature` | Environmental temperature; unit not documented |
| `humidity` | Relative humidity; unit not documented |
| `weight` | Weight or weight change; definition and unit not documented |
| `filesize` | Size of the associated image or capture file |
| `workers` | Image-derived worker-bee detections |
| `drones` | Image-derived drone detections |
| `ratio` | Derived ratio; exact formula not documented |
| `total` | Total bee detections; examples indicate `workers + drones` |

Stored descriptive statistics:

| Field | Mean | Standard deviation | Minimum | Median | Maximum |
| --- | ---: | ---: | ---: | ---: | ---: |
| Temperature | 90.115 | 4.619 | 71.420 | 91.400 | 99.680 |
| Humidity | 44.669 | 7.064 | 26.700 | 44.500 | 73.300 |
| Weight | 0.000513 | 0.003070 | 0 | 0 | 0.040920 |
| File size | 22,086,210 | 18,071,200 | 0 | 15,998,810 | 84,430,720 |
| Workers | 10.207 | 10.072 | 0 | 7 | 40 |
| Drones | 0.322 | 0.915 | 0 | 0 | 10 |
| Ratio | 2.000 | 5.787 | 0 | 0 | 37 |
| Total | 10.529 | 10.249 | 0 | 8 | 40 |

The raw null check reports zero missing values in every field. Zero values are common, particularly for weight and drone detections. One exploratory section temporarily treats zero drone detections as missing and drops those rows; the later modelling sections reload the source data and are not based on that filtered frame.

### Daily aggregation

The primary preprocessing path applies feature-specific daily aggregation:

| Feature | Daily operation |
| --- | --- |
| Temperature | Mean |
| Humidity | Mean |
| Weight | Last observation, renamed `weight_change` |
| File size | Sum |
| Workers | Sum |
| Drones | Sum |

This produces 195 daily rows. The notebook reports no missing values after this aggregation.

A separate random-forest path creates `daily_data_m2` with `dataset.resample('D').sum()`. In that frame, temperature and humidity are summed rather than averaged. This makes those values partly dependent on the number of observations recorded each day and prevents direct comparison with the primary daily frame.

## Worker-count regression

### Dataset and split

Target:

```text
workers
```

Features:

```text
temperature, humidity, filesize, drones, weight_change
```

The notebook uses a random 80/20 split with `random_state=0`:

- training rows: 156;
- test rows: 39;
- preprocessing: `MinMaxScaler` fitted on the training features.

Because this is time-series data, a random split can place future observations in the training set and earlier observations in the test set. The reported scores therefore do not represent a true forward-looking forecast.

### Linear regression

The first linear model is fitted on scaled training features but evaluated against the unscaled test matrix. Its stored output is:

| Metric | Recorded value |
| --- | ---: |
| MAE | 32,189,661,361.5 |
| MSE | 1.443 × 10²¹ |
| R² | -5.559 × 10¹⁷ |

These values are invalid as model evidence because training and evaluation use different feature scales. A later cell retrains linear regression after scaling both matrices, but it does not record evaluation metrics for that corrected model.

### Polynomial Ridge and Lasso

The scaled features are expanded to degree three with `PolynomialFeatures`, producing 56 columns.

Configuration:

| Model | Parameters |
| --- | --- |
| Ridge | `alpha=400`, `tol=0.0001` |
| Lasso | `alpha=3`, `tol=0.0001` |

Stored results:

| Model | Training R² | Test R² | Test MSE |
| --- | ---: | ---: | ---: |
| Polynomial Ridge | 0.278 | 0.322 | Not recorded |
| Polynomial Lasso | 0.829 | 0.836 | 425.847 |

The similar training and test scores for Lasso are encouraging within this split. They are not sufficient evidence of generalisation because:

- only one random split is used;
- no naive or seasonal baseline is reported;
- the split is not chronological;
- all rows originate from the same unidentified hive or source stream;
- the target is bee detections, not a health or intervention outcome.

## Total-count random forest

### Configuration

Target:

```text
total
```

Features:

```text
temperature, humidity, weight, filesize
```

Model and validation:

- `RandomForestRegressor`;
- 100 trees;
- `random_state=1234`;
- sequential groups of five daily rows;
- `GroupKFold(n_splits=10)`.

Grouping prevents the same five-day block from appearing in both sides of a fold. It does not enforce chronological training: a fold can still be evaluated using models trained on dates that occur after its test dates.

### Cross-validation results

| Fold | R² |
| ---: | ---: |
| 1 | 0.911 |
| 2 | 0.969 |
| 3 | 0.920 |
| 4 | 0.921 |
| 5 | 0.939 |
| 6 | 0.877 |
| 7 | 0.582 |
| 8 | 0.327 |
| 9 | 0.779 |
| 10 | 0.885 |
| **Mean** | **0.811** |

The range from `0.327` to `0.969` shows substantial variation between time groups. Reporting only the final fold's R² of `0.885` overstates the stability of the model; the mean and fold distribution are more informative.

The final stored fold reports:

| Metric | Value |
| --- | ---: |
| MAE | 15.034 detections |
| MSE | 351.517 |
| R² | 0.885 |

The notebook passes the target as a one-column DataFrame instead of a one-dimensional array. Scikit-learn emits a `DataConversionWarning` on every fit but still trains the model.

### Recorded feature importance

| Feature | Recorded importance |
| --- | ---: |
| File size | 0.920 |
| Humidity | 0.033 |
| Temperature | 0.026 |
| Weight | 0.021 |

This output requires two cautions.

First, file size may act as a proxy for image count, image content, capture duration, or a change in acquisition settings. Since the target is also derived from captured images, the relationship may be operational rather than biological.

Second, the implementation collects each tree's feature importance but then selects only the first tree from each fold before averaging. The table is therefore not the mean importance of the ten complete forests. Feature importance must be recalculated from each fitted forest, preferably with permutation importance on held-out data.

## Austrian dataset

Notebook: [`code/Time_series_analysis_austria.ipynb`](code/Time_series_analysis_austria.ipynb)

### Input and preprocessing

The source file is `bees_austrian_db.csv`. It contains 15,065 records with examples at 15-minute intervals.

Fields:

```text
date, temperature, humidity, windspeed, weight, winddir
```

Preprocessing replaces decimal commas, converts numeric fields with `errors='coerce'`, and writes an intermediate `converted_data.csv`. Invalid numeric values consequently become `NaN`.

Daily aggregation takes the mean of temperature, humidity, wind speed, wind direction, and weight. The notebook then drops weight before analysis and modelling. The resulting frame contains 392 calendar-day rows.

### Exploratory analysis

The notebook includes:

- a box plot across daily variables;
- a temperature histogram;
- time-series plots for temperature, humidity, wind direction, and wind speed.

The missing-value check uses `daily_data.isnull().count()`. That expression returns the number of rows per column, not the number of missing values. The displayed value of `392` for each field is therefore a row count, not a null count. The correct check is `daily_data.isnull().sum()`.

### Model attempt

Target:

```text
temperature
```

Features:

```text
humidity, windspeed, winddir
```

The notebook creates a random 80/20 split with `random_state=42` and attempts to fit a 100-tree `RandomForestRegressor`. Training stops with:

```text
ValueError: Input X contains NaN.
```

No imputation, row filtering, missingness indicators, or time-aware split is applied. No valid Austrian model metric is produced.

This experiment also does not target hive health: it estimates temperature from other weather variables after dropping hive weight.

## What the results support

The notebooks demonstrate that:

- heterogeneous sensor and image-derived records can be converted into daily analytical features;
- worker and total detection counts contain predictable structure within the American dataset;
- grouped evaluation produces positive R² values, although performance varies strongly by fold;
- missing-data handling is a first-order requirement for a multi-source monitoring pipeline.

They do not demonstrate that:

- the models detect disease, queen loss, swarming, starvation, or another health event;
- performance transfers to a different hive, apiary, geography, camera, or season;
- an alert would improve a beekeeper's decision;
- the reported accuracy survives a chronological or external validation set;
- image file size is a stable or commercially usable predictor.

## Main validity risks

1. **No health labels.** The targets are bee detections and temperature, not operational outcomes.
2. **No hive identifier.** The notebooks cannot measure generalisation to unseen hives.
3. **Temporal leakage.** Random splits and non-chronological grouped folds can train on later observations.
4. **Acquisition leakage.** File size is closely tied to the image pipeline that creates the count target.
5. **Inconsistent aggregation.** One model uses daily means for environmental variables; another uses sums.
6. **Unresolved missing data.** The Austrian model fails before training.
7. **Undocumented units and schema.** Several features cannot be interpreted confidently.
8. **Stateful notebooks.** Frames are mutated and reloaded across cells, so results depend on execution order.
9. **No fixed environment.** Dependency versions are not pinned.
10. **Private input data.** Reproduction is limited to authorised holders of the source files.

## Recommended validation design

A credible next experiment should start with the decision being supported, not with a model choice.

### 1. Define the target

Record explicit outcomes such as:

- inspection required within a defined time window;
- queen loss;
- swarm event;
- abnormal weight loss;
- feeding requirement;
- confirmed disease or pest event.

Each label should include event time, severity, intervention, and beekeeper confirmation.

### 2. Define the data contract

For every observation, retain:

```text
hive_id, apiary_id, timestamp, sensor_id, units, sampling status, firmware or capture configuration
```

Distinguish true zero values from missing measurements and record changes in camera or sensor settings.

### 3. Establish baselines

Compare every model with simple alternatives:

- previous-day value;
- rolling mean or seasonal baseline;
- threshold rules already used by beekeepers;
- a model using only observation count or file size.

The final comparison is necessary to determine whether biological features add value beyond the capture process.

### 4. Validate by hive and time

Use earlier periods for training and later periods for testing. Hold out complete hives or apiaries for external validation. Report performance by season, site, device configuration, and event type.

### 5. Measure operational value

Regression metrics are not enough for an alerting product. Report:

- event recall;
- false alerts per hive per month;
- median warning time before inspection;
- inspections avoided;
- losses or interventions associated with acted-on alerts.

A field trial should define acceptable thresholds with beekeepers before evaluation.

## Reproduction notes

Place authorised source files in `code/`:

```text
bee_data.csv
bees_austrian_db.csv
```

Create and activate a Python environment, then install the unpinned dependencies used by the notebooks:

```bash
python -m pip install jupyter pandas numpy matplotlib seaborn scikit-learn statsmodels openpyxl
cd code
jupyter notebook
```

The current notebooks should be treated as exploratory records rather than a reproducible pipeline. Before rerunning them as an experiment, fix the scaling mismatch, aggregation inconsistency, null check, Austrian missing-data handling, target shape, temporal validation, and feature-importance calculation described above.
