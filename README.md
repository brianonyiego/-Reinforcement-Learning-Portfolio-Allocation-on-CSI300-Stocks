# Reinforcement Learning Portfolio Allocation on CSI300 Stocks

This repository builds a dense daily price panel of 30 Chinese A-share stocks from a Bloomberg export of CSI300 constituents, and uses it as the investment universe for a reinforcement-learning (RL) portfolio allocation study.

This README covers the **data pipeline** (report sections 4.1 and 4.2) and the **regime analysis check**. Fill in the `TODO` items for the parts of the project that are not visible from the console output alone.

---

## Contents

- [Overview](#overview)
- [Dataset summary](#dataset-summary)
- [Repository structure](#repository-structure)
- [Requirements](#requirements)
- [Input data format](#input-data-format)
- [How to run](#how-to-run)
- [What the pipeline does](#what-the-pipeline-does)
- [Example output](#example-output)
- [Regime analysis](#regime-analysis)
- [Limitations](#limitations)
- [Configuration](#configuration)
- [Troubleshooting](#troubleshooting)

---

## Overview

| Step | Report section | Description |
|------|----------------|-------------|
| 1 | 4.1 | Read the Bloomberg workbook, parse one block per ticker, select the 30 most complete tickers, forward/back-fill gaps and build a dense panel. |
| 2 | 4.2 | Print and save a dataset description (source, universe, tickers, sample period, frequency, train/test split). |
| 3 | 4.x | Check whether the configured market-regime windows overlap the test set, and plot the ones that do. |

---

## Dataset summary

| Item | Value |
|------|-------|
| Data source | Bloomberg export (CSI300 constituents) |
| Raw universe | 300 tickers, about 1.09 million raw rows |
| Selected universe | 30 Chinese A-share stocks, chosen by data completeness |
| Completeness of selected tickers | 0.999 to 1.000 (fraction) |
| Sample period | 2011-06-16 to 2026-06-15 |
| Frequency | Daily |
| Final dense panel | 3,642 common trading days x 30 tickers |
| Test set | 2023-06-12 to 2026-06-15 |
| Filled cells | 556 cells across 30 tickers were forward/back-filled |

### Selected tickers

```
601601 CH, 600015 CH, 601607 CH, 601668 CH, 600276 CH, 000596 CH,
601288 CH, 600111 CH, 601988 CH, 600600 CH, 600519 CH, 600085 CH,
600011 CH, 601398 CH, 600660 CH, 601898 CH, 601939 CH, 601857 CH,
600028 CH, 601628 CH, 002028 CH, 002304 CH, 601328 CH, 600016 CH,
601006 CH, 002463 CH, 600009 CH, 000338 CH, 601318 CH, 601117 CH
```

---

## Repository structure

> TODO: adjust to match your actual files.

```
.
├── README.md
├── requirements.txt
├── data/
│   └── CSI300_constituents.xlsx      # Bloomberg export (not committed)
├── src/
│   ├── data_builder.py               # Section 4.1: build raw dataset / dense panel
│   ├── dataset_description.py        # Section 4.2: dataset description
│   ├── regimes.py                    # Regime windows and test-set overlap check
│   └── ...                           # TODO: RL environment, agents, evaluation
├── outputs/
│   └── ...                           # panels, tables, figures
└── main.py                           # TODO: entry point
```

---

## Requirements

- Python 3.9+
- `pandas`
- `numpy`
- `openpyxl` (reads the `.xlsx` Bloomberg workbook)
- `matplotlib` (regime plots)
- TODO: RL libraries (for example `torch`, `gymnasium`, `stable-baselines3`)

Install:

```bash
python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate
pip install -r requirements.txt
```

---

## Input data format

The pipeline expects a Bloomberg Excel export, for example `CSI300_constituents.xlsx`, where **each ticker occupies its own block of columns** (a date column plus one column per field, such as price). Blocks for all 300 constituents sit side by side in the same workbook.

- Empty or blank blocks are skipped (the log reports how many).
- The universe file name (without extension) is used as the universe label, for example `CSI300`.
- Several universe files can be supplied; the log lists them under `Universe file(s)`.

> TODO: list the exact fields you read (for example `PX_LAST`, `PX_VOLUME`) and the expected header layout.

The Bloomberg data are licensed and **are not included** in this repository. Place your own export in the data folder and set the path in the configuration (see below).

---

## How to run

```bash
python main.py
```

> TODO: replace with your real command(s) or notebook name.

Console output is organised by report section (see the example below), so it can be copied directly into the write-up.

---

## What the pipeline does

### 4.1 Building the raw dataset

1. **Read** each workbook in the data folder.
2. **Parse** one block per ticker, skipping empty blocks, and stack them into a long raw table.
3. **Select** the top 30 tickers by data completeness (fraction of non-missing observations).
4. **Fill gaps** with forward fill, then back fill. Gaps are mostly trading suspensions or export gaps. The number of filled cells is logged.
5. **Build a dense panel** on the common trading days shared by all selected tickers (3,642 days x 30 tickers).

### 4.2 Dataset description

Prints the data source, universe, tickers, sample period, frequency, and the train/test split, including the test-set date range.

### Regime check

Compares each window in `REGIME_PERIODS` with the test-set dates and plots only those that overlap. If none overlap, it prints a message and suggests an ad hoc window.

---

## Example output

```
==============================================================================
4.1 BUILDING RAW DATASET FROM BLOOMBERG WORKBOOKS
==============================================================================
  Reading C:\Users\<user>\Downloads\Data\CSI300_constituents.xlsx ...
    Parsed 300 tickers, skipped 0 empty/blank blocks, 1,092,900 raw rows

  Universe file(s): ['CSI300']
  Total tickers parsed: 300
  Selected top 30 tickers by data completeness for the RL universe.
  Completeness range of selected tickers: 0.999 - 1.000 (fraction, decimal form)
  ...
  Final dense panel: 3,642 common trading days, 30 tickers

==============================================================================
4.2 DATASET DESCRIPTION
==============================================================================
Data source            : Bloomberg export (CSI300)
Stock universe         : 30 Chinese A-share stocks (fixed ex-post universe)
Sample period          : 2011-06-16 to 2026-06-15
Frequency              : Daily
...   Test set covers: 2023-06-12 to 2026-06-15
```

---

## Regime analysis

Configured market regimes:

| Regime | Start | End | Overlaps test set |
|--------|-------|-----|-------------------|
| Bull (2016-2017) | 2016-01-01 | 2017-12-31 | No |
| Bear (2018) | 2018-01-01 | 2018-12-31 | No |
| COVID (2020 H1) | 2020-01-01 | 2020-06-30 | No |
| Post-COVID (2021-22) | 2021-01-01 | 2022-12-31 | No |

**None of the configured regimes fall inside the test window** (2023-06-12 to 2026-06-15), so the regime plot is skipped. To analyse the test period, define a window inside it:

```python
REGIME_PERIODS_ADHOC = {"Test period": ("2023-06-12", "2026-06-15")}
```

You can also split the test set into sub-periods (for example by market drawdown or volatility) or evaluate regimes on the training/validation period instead.

---

## Limitations

**Survivor bias (fixed ex-post universe).** The constituent list reflects index membership at the time of the Bloomberg export, projected backward over the whole sample. Stocks removed from the index earlier (delisted, acquired, or dropped for poor performance) are excluded. This is **not** a point-in-time constituent list, and it biases the sample toward stocks that survived. Point-in-time constituent data were not available. The study should be described as *a fixed ex-post universe of 30 CSI300 constituent stocks selected based on data completeness*, and any absolute-return conclusions should be read as an **upper bound**, not a fully realistic, investable backtest.

**Filled data.** 556 cells across 30 tickers were forward/back-filled. Suspension days therefore carry a stale price instead of being dropped from the panel, which can understate volatility and distort returns around suspensions.

**Selection on completeness.** Choosing tickers by data completeness favours long-listed, liquid, large-cap names (for example banks, insurers, energy and large consumer stocks).

---

## Configuration

> TODO: match these to your real variable names.

| Setting | Description | Value |
|---------|-------------|-------|
| `DATA_DIR` | Folder containing the Bloomberg workbook(s) | `C:\Users\<user>\Downloads\Data` |
| `N_TICKERS` | Number of tickers kept by completeness | `30` |
| `TEST_START` / `TEST_END` | Test-set date range | `2023-06-12` / `2026-06-15` |
| `REGIME_PERIODS` | Named regime windows for plots | see [Regime analysis](#regime-analysis) |

---

## Troubleshooting

- **File not found:** check `DATA_DIR` and use raw strings or forward slashes for Windows paths.
- **Fewer than 300 tickers parsed:** check for empty or merged header cells in the workbook; blank blocks are skipped silently but counted in the log.
- **No regime plots:** expected when no regime overlaps the test set. Add an ad hoc window as shown above.
- **Duplicate console output:** if sections 4.1 and 4.2 print twice, the pipeline is probably being called twice (for example once at import and once in `main`). Guard it with `if __name__ == "__main__":`.

---

## Disclaimer

This project is for academic research only and is not investment advice. Results are based on a survivor-biased universe and should not be treated as an investable strategy.
