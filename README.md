# Evaluation of COVID-19 Models

**June 27 Update:** We have added [global](/global) evaluations comparing the *YYG / covid19-projections.com* and *IHME* models with the baseline. See more details [below](#global-evaluation).

Here we present an evaluation of models from the [COVID-19 Forecast Hub](https://github.com/reichlab/covid19-forecast-hub). These models are submitted weekly to the [CDC COVID-19 Forecasting page](https://www.cdc.gov/coronavirus/2019-ncov/covid-data/forecasting-us.html) to help inform public health decision-making.

While a model's future projections can be useful, it is also important to take into account the model's historical performance in a transparent, rigorous, and non-biased manner. This is the goal of this project.

**Evaluations are done weekly and summarized in the [summary](/summary) directory.** You can view the outputs of the individual evaluations in the [evaluations](/evaluations) directory.

**Table of Contents**
* [Dependencies](#dependencies)
* [Getting Started](#getting-started)
* [Usage](#usage)
  * [Evaluation](#evaluation)
  * [Summary](#summary)
* [Details](#details)
  * [Overview](#overview)
  * [Models and Teams](#models-and-teams)
  * [Methods](#methods)
  * [US Evaluation](#us-evaluation)
  * [State-by-state Evaluation](#state-by-state-evaluation)
  * [Baseline Model](#baseline-model)
  * [Global Evaluation](#global-evaluation)

## Dependencies

### Python

You need Python 3 with the [NumPy](https://numpy.org/install/) and [pandas](https://pandas.pydata.org/getting_started.html) packages.

Once you have Python 3, you can simply install the NumPy and pandas packages by running the following command:
```
pip install numpy pandas
```

### Data

You also need to download the [COVID-19 Forecast Hub](https://github.com/reichlab/covid19-forecast-hub) data, which contains the raw forecasts from each model. We recommend cloning the repository:
```
git clone https://github.com/reichlab/covid19-forecast-hub.git
```

## Getting Started
1. Make sure the dependencies (NumPy/pandas) are installed: `pip install numpy pandas`
2. Clone this repository: `git clone https://github.com/youyanggu/covid19-forecast-hub-evaluation.git`
3. Clone the [COVID-19 Forecast Hub](https://github.com/reichlab/covid19-forecast-hub) repository: `git clone https://github.com/reichlab/covid19-forecast-hub.git`
4. *(Optional)* Make sure that the COVID-19 Forecast Hub and this repository share the same parent. Otherwise, you would need to pass in the Forecast Hub location via the `--forecast_hub_dir` flag.
5. Pick a Monday for the projection date (when the forecasts were generated), and pick a Saturday to evaluate those projections (e.g. Monday May 4 and Saturday June 13). We explain the reasoning for this [below](#details).
6. Run the evaluation: `python evaluate_models.py 2020-05-04 2020-06-13`. You can view a list of sample commands [here](evaluation_cmds.txt).
7. *(Optional)* After you have generated several evaluations, you can summarize the evaluations: `python summarize_evaluations.py --weeks_ahead 4`. You can view a list of sample commands [here](summary_cmds.txt).

## Usage

### Evaluation

We call `evaluate_models.py` to compute an evaluation based on a projection date-evaluation date pair. Note that the projection date must be a Monday and the evaluation date must be a Saturday due to reasons [explained below](#details).

#### Evaluate Mon May 4 projections on Sat June 13 data
```
python evaluate_models.py 2020-05-04 2020-06-13
```

#### Custom COVID-19 Forecast Hub directory
By default, it checks the local directory for `covid19-forecast-hub`.
```
python evaluate_models.py 2020-05-04 2020-06-13 --forecast_hub_dir /PATH/TO/covid19-forecast-hub
```

#### Save evaluation results to a directory
```
python evaluate_models.py 2020-05-04 2020-06-13 --out_dir evaluations/
```

#### Use median projections rather than point projections
For most models, this makes no difference.
```
python evaluate_models.py 2020-05-04 2020-06-13 --use_median
```

#### Print additional statistics such as mean rank and residual analysis
```
python evaluate_models.py 2020-05-04 2020-06-13 --print_additional_stats
```

#### Run weekly evaluation for all models since April 20
This is the command that generated all of the files in the `evaluations` directory.
```
. evaluation_cmds.txt
```

### Summary

We call `summarize_evaluations.py` to summarizes the individual evaluations generated from above.

#### Summarize all projections with 4 weeks ahead forecasts
```
python summarize_evaluations.py --weeks_ahead 4
```

#### Summarize all projections ending on Sat June 13
```
python summarize_evaluations.py --eval_date 2020-06-13
```

#### Custom evaluations directory
The evaluations directory is the output of the `evaluate_models.py` script (default is simply the `evaluations/` directory).
```
python summarize_evaluations.py --weeks_ahead 4 --evaluations_dir /PATH/TO/evaluations
```

#### Save evaluation results to a directory
```
python summarize_evaluations.py --weeks_ahead 4 --out_dir summary/
```

#### Summarize all weekly evaluations since April 20
This is the command that generated all of the files in the `summary` directory.
```
. summary_cmds.txt
```

## Details

### Overview

Models in the COVID-19 Forecast Hub submit their forecasts every Monday to be sent to the CDC. To be declared valid, a model must have 1-4 week ahead forecasts. For these week-ahead forecasts, all models use the specification of [epidemiological weeks (epiweek)](https://wwwn.cdc.gov/nndss/document/MMWR_Week_overview.pdf) defined by the CDC. Because each epiweek ends on Saturday, all models are also evaluated on Saturdays. For example, if a model submits a forecast on Monday, June 15, a one week ahead forecast corresponds to the forecast ending on Saturday, June 20. A two week ahead forecast corresponds to Saturday, June 27, and so forth. This is explained in more detail in the [COVID-19 Forecast Hub README](https://github.com/reichlab/covid19-forecast-hub/#covid-19-forecast-hub).

To summarize, due to the reason above, we standardize all projection dates to be on Mondays and all evaluation dates to be on Saturdays.

### Models and Teams

An entire list of [models and teams](https://github.com/reichlab/covid19-forecast-hub/#teams-and-models) is presented on the COVID-19 Forecast Hub page. For more details on an individual team's model, you can look for the metadata file in the `data-processed` directory of the Forecast Hub (example: [COVIDhub Ensemble metadata](https://github.com/reichlab/covid19-forecast-hub/blob/master/data-processed/COVIDhub-ensemble/metadata-COVIDhub-ensemble.txt)).

The [COVIDhub ensemble model](https://github.com/reichlab/covid19-forecast-hub/#ensemble-model) is a model created by the Reich Lab that takes a combination of all models that submit eligible projections to the Forecast Hub. You can see which models are included and their corresponding weights [here](https://github.com/reichlab/covid19-forecast-hub/tree/master/ensemble-metadata).

When doing evaluations I think it's important to consider only one model per team. If a team submits 10 different models each with a different forecasts, then they will undoubtedly have a higher chance of having a more accurate model compared to a team that only submits a single forecast. That's why the COVID-19 Forecast Hub asks every team to designate only a single model to include in forecasts sent to the CDC. And so we are only consider that particular model in the evaluation.

### Truth Data

As described in the [COVID-19 Forecast Hub README](https://github.com/reichlab/covid19-forecast-hub/tree/master/data-processed#ground-truth-data), all forecasts are compared to the [Johns Hopkins University CSSE Time Series Summary](https://github.com/CSSEGISandData/COVID-19/tree/master/csse_covid_19_data/csse_covid_19_time_series) as the gold standard reference data for deaths in the US. This truth data can be found in the [Forecast Hub data-truth directory](https://github.com/reichlab/covid19-forecast-hub/tree/master/data-truth).

### Methods

For simplicity purposes we will be comparing the [point forecast](https://github.com/reichlab/covid19-forecast-hub/tree/master/data-processed#type) to the truth data described above. The point forecast is almost always the mean or median estimate of the model.

There are [more advanced](https://arxiv.org/pdf/2005.12881.pdf) techniques for scoring forecasts that take advantage of [quantile estimates](https://github.com/reichlab/covid19-forecast-hub/tree/master/data-processed#quantile), but we will leave that as a future extension.

### US Evaluation

For US country-wide forecasts, we compute the error for each model's point forecasts using the following formula:
```
additional_deaths = true_deaths_eval_date - true_deaths_day_before_proj_date
error_us = predicted_deaths_eval_date - true_deaths_eval_date
perc_error_us = error_us / additional_deaths
```
where `true_deaths` is the actual cumulative deaths, `predicted_deaths` is the forecasted cumulative deaths, `eval_date` is the date when the forecasts of total deaths are being evaluated against, and `proj_date` is the date the projection was generated.

So for example, if our projection date is Monday, June 1 and our evaluation date is Saturday, June 13, below is a sample of how we compute the error.

| Example | |
| --- | --- |
| May 31 US true deaths | 104,659 |
| June 13 US true deaths | 115,436 |
| Additional deaths | 10,006 (115,436 - 105,430) |
| Sample model forecast for June 13 | 115,581 |
| Error | 145 (115,581 - 115,436) |
| % Error | 1.4% (145 / 10,006) |

### State-by-state Evaluation

In addition to US country-wide forecasts, we also evaluate state-by-state forecasts for all 50 states plus DC. In this section, we only evaluate models that have 1-4 week ahead forecasts for more than 40 states.

Using the point forecast for every model, we compute the error for each state by the following:

```error_state = projected_deaths_eval_date - true_deaths_eval_date```

For example, if a state reports 2,462 deaths on June 13 and the model's forecasted total for June 13 is 2,420, then our error for that state is -42.

We then compute the mean absolute error and mean squared error for all states for every model. For models with missing state projections, we substitute the error with the mean absolute error for that state (among all the models).

### Baseline Model

In any evaluation, it is important to include a baseline as a control, similar to how scientfic trials include a placebo. We define a simple baseline model that takes the mean of the previous week's daily deaths to make all future forecasts. For example, for Monday, May 25 projections, we use the average daily deaths from May 18 to May 24 to make forecasts. For US country-wide projections, this would amount to a constant 1,164 deaths per day for each forecast day.

We also include another baseline model that takes mean of the previous week's daily deaths and decrease that by 2% each day for all future projections (called `Baseline_0.98`). This is in general a much more accurate model, but may be subject to selection bias.

### Global Evaluation

In addition to US forecast evaluations, we also include evaluations of country-by-country forecasts by [covid19-projections.com](https://covid19-projections.com) and [IHME](https://covid19.healthdata.org/). Because global forecasts are not standardized, each team's forecasts must be downloaded and processed separately. We compare the models with the baseline model (explained above). You can view the results in the [`global`](/global) directory.

## Questions? Bugs? Feature Request?

Lastly, we encourage open collaboration. Please open [an issue request](https://github.com/youyanggu/covid19-forecast-hub-evaluation/issues) if you have any questions/suggestions/bug reports.
