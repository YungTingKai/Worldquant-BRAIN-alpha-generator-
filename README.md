# WorldQuant BRAIN Alpha Generator

 

A Python/Jupyter toolkit that automates part of the alpha research workflow on the [WorldQuant BRAIN](https://platform.worldquantbrain.com/) platform: authenticate via the BRAIN API, pull a dataset's data fields, expand them into many candidate alpha expressions using reusable templates, and batch-submit them for simulation.

 

The goal is to replace manually writing and submitting one alpha expression at a time with a small set of templates that get automatically combined with data fields, operators, lookback windows, and grouping schemes — scaling idea generation instead of idea-by-idea authoring.

 

## How it works

 

1. **Authenticate** — log in to the BRAIN API with `requests` + `HTTPBasicAuth`, reuse the session for all subsequent calls (`day2.ipynb`, `day3.ipynb`).

2. **Pull data fields** — query the `/data-fields` endpoint (paginated) for a given dataset (e.g. `fundamental6`, Company Fundamental Data for Equity), filtered by region / universe / instrument type, and load the results into a pandas DataFrame.

3. **Generate alpha expressions from a template** — substitute each data field (and, in the later version, each operator/lookback/grouping combination) into a fixed expression template:

- `day2.ipynb`: a single template, `group_rank(<datafield>/cap, subindustry)`, applied to every matrix-type field in `fundamental6` → 574 alpha expressions generated automatically.

- `day3.ipynb`: a combinatorial template, `<group_op>(<ts_op>(<datafield>, <days>), <group>)`, crossed across 3 group-level operators (`group_rank`, `group_zscore`, `group_neutralize`) × 3 time-series operators (`ts_rank`, `ts_zscore`, `ts_quantile`) × all fundamental data fields × 2 lookback windows (600/200 days) × 4 grouping schemes (`market`, `industry`, `subindustry`, `sector`) → thousands of expressions from one template definition.

4. **Batch-submit for simulation** — POST each alpha to `/simulations`, poll the returned `Location` URL until the backtest finishes, and collect the resulting alpha ID.

5. **Resilience (added in `day3.ipynb`)** — wrap each submission in a retry loop with a failure-count tolerance; if a request keeps failing, the session re-authenticates and moves on to the next alpha. All attempts and errors are written to a log file (`simulation_day3.log`) instead of only printing to stdout.

 

## Files

 

| File | Description |

|---|---|

| `day2.ipynb` | First version: single fixed template applied to one dataset's fields (~574 alphas). |

| `day3.ipynb` | Expanded version: multi-operator / multi-window / multi-group combinatorial template generation, plus retry logic, automatic re-login, and logging. |

| `day4.ipynb` | Further iteration on the generation/submission workflow. |

 

## Tech stack

 

Python, `requests`, `pandas`, `logging`, Jupyter Notebook, WorldQuant BRAIN REST API.

 

## Setup

 

```bash

pip install requests pandas

```

 

Create a `brain_credentials.txt` file in the working directory containing your BRAIN login as a JSON list: `["username", "password"]`. This file is read at runtime and is **not** committed to the repository.

 

## Status

 

This is a research-automation tool, not a curated alpha library. It is used to generate and submit large batches of candidate alphas for simulation; quantitative results (Sharpe, IS/OOS performance, pass rates) depend on which simulations are run and are not reported here.

 

## Disclaimer

 

Built for personal research and learning on the WorldQuant BRAIN platform. Not affiliated with or endorsed by WorldQuant.

 
