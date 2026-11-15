# Caffeine Forecast

Does each American generation drink more caffeine than the last?
25 years of CDC survey data say no.

## The question

I started with an assumption: that caffeine consumption escalates generation
over generation, with each cohort starting younger and drinking more. The data
contradicted it.

Across 11 NHANES survey cycles covering 57,000 US adults from 1999 to 2023,
caffeine intake falls by roughly 3.25 mg per day for every later birth year,
measured at the same age. Boomers averaged 208 mg a day at 35. Someone born in
the late 1990s is on track for 120 mg at that age, about 42% less. Projected
forward, per-capita intake across US adults drops 11.7% between 2024 and 2035.

That reframes the project. This is not escalating dependence. It is structural
category decline, which is a demand problem for anyone selling caffeine.
## Key findings

- **Intake falls about 3.25 mg/day for every later birth year**, measured at
  the same age (survey-weighted, R² 0.62). The effect holds independently in
  the 20s, 30s, 40s and 50s, so it is not an artifact of one life stage.
- **At age 35: Boomers 208 mg, Gen X 183, Millennials 144, Gen Z 120.** Roughly
  two and a half cups of coffee down to one and a quarter.
- **Heavy users are disappearing.** The share exceeding the FDA's 400 mg daily
  limit halves with each generation: Boomers peaked at 21%, Gen X holds near
  10%, Millennials sit at 5%, Gen Z under 1%. Millennials have shown no sign of
  rising toward Gen X levels as they age, so the segment is not being replaced.
- **Per-capita intake is projected to fall 11.7% between 2024 and 2035**, before
  accounting for the fact that the heaviest-consuming cohorts are also shrinking
  in number.

## How it works

| Notebook | What it does |
|---|---|
| `01_ingest` | Builds 22 CDC download URLs from a cycle-to-suffix map, fetches them idempotently, and writes a provenance log |
| `02_explore` | Loads one cycle, locates the variables, validates the weighted mean against published figures |
| `03_clean` | Probes every file for schema drift, standardises 11 cycles into one table of 57,459 adults, runs the cohort analysis |
| `04_forecast` | Fits and validates the model on held-out cycles, projects to 2035 |

## Validation

The pipeline is checked against external ground truth rather than its own
internal consistency.

- **Benchmark.** Published NHANES analyses put US adult intake at 169 mg/day.
  This pipeline produces 166.6 mg for 2017-18, a 1.4% deviation. The first
  attempt gave 141 mg, which is how I found I was using no survey weights.
  Dietary analyses require `WTDRD1`, not the exam weight.
- **Holdout.** The model trains on cycles through 2015-16 and is tested on
  2017-18 and 2021-23, real unseen future data rather than a random split.
  Cohort-level error is 16.7 mg, or 11.6%.
- **Model comparison.** Gradient boosting was tested and lost to the linear
  model, as expected: a time-based split tests extrapolation, and tree models
  predict constants beyond their training range.
- **A model I did not ship.** A logistic model for the share above 400 mg
  failed validation at 64% relative error and contradicted observed Millennial
  behaviour, so no threshold forecast is published.

## Limitations

- Association, not causation. Cross-sectional survey data.
- Age, period and cohort cannot be separately identified. A secular shift
  affecting everyone would be attributed to cohort here.
- The 2035 projection extrapolates past the fitted range of birth years.
- Single-day 24-hour recall, self-reported. The second recall day exists from
  2003 and is not used.
- Residual error is largest above age 70; the forecast is most reliable
  between 20 and 60.

## Reproducing

```bash
git clone https://github.com/HargunKohli/caffeine-forecast
cd caffeine-forecast
python3 -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
```

Run the notebooks in order. `01_ingest` re-downloads the raw data from the CDC,
so nothing depends on files living on my machine.

## Dashboard

*Coming soon.*