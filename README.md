# Beehive Analytics

Decision-support research for data-driven beehive monitoring.

- **Status:** research prototype
- **Technical report:** [Read the full results and methodology](results.md)

This project was developed through a collaboration between Howest University of Applied Sciences and the University of Burgos. It tests whether sensor readings and image-derived bee counts can help operators decide which hives need attention, rather than inspecting every hive on the same schedule.

## The problem

Hive inspections are manual, time-consuming, and difficult to prioritise across multiple sites. Existing sensors can collect temperature, humidity, weight, wind, and image data continuously, but raw measurements do not tell an operator what action to take.

The commercial opportunity is an operational layer between hive sensors and field teams: identify unusual changes, rank inspections, and provide evidence for intervention decisions.

## Product thesis

A useful product would combine existing hive telemetry with a simple workflow:

1. ingest measurements from sensors and cameras;
2. establish a baseline for each hive and season;
3. flag meaningful deviations;
4. rank hives by inspection priority;
5. record inspection outcomes to improve future alerts.

The current repository covers the analytical first step. It does not yet include live ingestion, alerts, a dashboard, or a validated health-risk model.

## Commercial hypothesis

| Area | Hypothesis to validate |
| --- | --- |
| Initial customer | Commercial apiaries and pollination operators managing enough hives for inspection planning to be costly |
| Buyer | Owner, operations manager, or head beekeeper |
| Core value | Fewer low-value inspections and earlier attention to hives showing abnormal behaviour |
| Product entry point | Analytics and inspection prioritisation on top of existing sensor deployments |
| Revenue model | Recurring fee per monitored hive or apiary |
| Main adoption risk | Alerts must be reliable enough to change field decisions without creating additional work |

These are product hypotheses, not conclusions from completed customer discovery.

## Evidence so far

The project contains two exploratory datasets:

- **United States:** 1,355 observations combining environmental measurements with worker and drone detections, aggregated into 195 daily records.
- **Austria:** 15,065 environmental sensor readings, aggregated into 392 daily records.

The American analysis found enough statistical signal to justify a better-designed validation study. Its strongest recorded model averaged an R² of `0.811` across grouped folds, but much of that signal appears tied to image file size. That may represent the image-capture process rather than colony health. The Austrian experiment exposed missing-data issues and did not produce a valid model result.

The practical conclusion is narrow: the data pipeline is feasible, but the project has not yet demonstrated that it can predict a health event or improve an inspection decision. See [`results.md`](results.md) for the models, metrics, implementation issues, and interpretation.

## What must be proven next

Before building a product, the project needs:

- explicit labels for health events and beekeeper interventions;
- data from multiple hives, sites, and seasons;
- validation on hives the model has never seen;
- agreed alert thresholds tied to the cost of missed and false alerts;
- customer evidence that better inspection prioritisation saves time or prevents losses;
- confirmed rights to use the required data commercially.

The next milestone should be a labelled field trial, not a larger model.

## Repository

```text
code/       Jupyter notebooks for the American and Austrian analyses
images/     Exploratory charts
results.md  Detailed methodology, metrics, and limitations
```

The raw datasets are not included. Authorised users can place `bee_data.csv` and `bees_austrian_db.csv` in `code/` and run the notebooks locally.

## Data and licensing

The American dataset was supplied by Appalachian State University for use in this project and cannot be redistributed here. The Austrian source data is also excluded from the repository.

The source code and notebooks are available under the [MIT License](LICENSE). That license does not grant rights to the underlying datasets.
