# Food Delivery Lab: Beginner's Guide

This guide explains what the assignment is, how I solved it, what you need to run it yourself, and what each answer means. You don't need prior knowledge.

## 1. The assignment

> "Solve the tasks in `notebook.ipynb`. Upload the completed `notebook.ipynb`."

`notebook.ipynb` is a **Jupyter notebook**: a document made of *cells*. Some cells are text (instructions) and some are Python code you run. The notebook is **already completed** and run from top to bottom, with all outputs and figures saved inside it. Upload that file.

The topic is **data science**. A fictional company, *City Delivery*, has a small website with 192 food-delivery orders. The lab asks you to:

1. **Scrape** the data from the website (read it automatically from web pages),
2. **Clean and describe** it (types, missing values, units, charts),
3. **Test hypotheses** with statistics (is the advertised delivery time true? does rain slow deliveries?).

## 2. What's in this folder

| Path | What it is |
|---|---|
| `notebook.ipynb` | The notebook, **solved**. This is the file to upload. |
| `GUIDE.md` | This guide. |
| `requirements.txt` | The Python packages the notebook needs. |
| `data/site/` | A saved copy of the website (`index.html`, `receipts.html`, `catalogue/page-1..16.html`). |
| `data/site.source.yaml` | Says the data is synthetic and free to use (CC0). |
| `.venv/` | A Python environment I created with all packages installed. It is only for running the notebook. You don't upload it. |

The notebook starts a tiny web server on your own computer (address `127.0.0.1`) that serves `data/site`. Your code then "visits" the pages like a browser. It **needs no internet** and nothing leaves your machine.

## 3. How to set it up yourself

You need **Python 3.10 or newer**. It was tested with 3.14.

```bash
cd ~/Downloads/02-food-delivery

# 1. create an isolated environment (already done here, so you can skip it)
python3 -m venv .venv
source .venv/bin/activate

# 2. install the packages
pip install -r requirements.txt

# 3. open the notebook
jupyter lab notebook.ipynb      # or: jupyter notebook notebook.ipynb
```

In Jupyter choose **Kernel > Restart & Run All**. If the kernel is not found, pick the Python from `.venv`. You can also open the notebook in **VS Code** (Python and Jupyter extensions) and select the `.venv` interpreter.

To run it without opening a window:

```bash
.venv/bin/jupyter nbconvert --to notebook --execute notebook.ipynb --output notebook.ipynb
```

**Before you submit:** run it top to bottom once more so the outputs are fresh. The lab says the notebook must run from top to bottom without errors and without internet.

### The packages, in one line each

- **pandas**: tables (`DataFrame`) and data cleaning.
- **numpy**: fast numbers and random sampling.
- **requests**: downloads a web page.
- **beautifulsoup4**: picks pieces out of HTML (titles, ratings...).
- **scipy / statsmodels**: statistical tests.
- **matplotlib / seaborn**: charts.
- **playwright**: a browser robot. It is imported by the setup cell but **not needed** for my solution (see Task 2).

## 4. The tasks and my solutions

### Task 1: Collect the orders
The orders are on 16 pages, linked by a "Next" button. The code downloads `page-1.html`, reads every order card, follows "Next" and repeats until there is no next page. Each card gives restaurant, cuisine, city, distance, time, slot, weather, payment, rating and who delivered.

Details:
- A missing delivery time or rating means that line is not on the card, so I store a **missing value**, not 0.
- Distances look like `5.0 km` or `3 650 m`. I store the number and the unit separately, exactly as quoted.

**Result: 192 orders over 16 pages.**

### Task 2: Add the receipts
`receipts.html` looks empty because JavaScript fills the table after the page loads. The data is already in the page source as a list (`const RECEIPTS = [...]`), so I read that list directly. No browser is needed.

The page holds **193 receipts**, but one order (`510546`) appears twice with identical values. I removed the extra copy, then joined receipts to orders by `order_id`.

**Result: `df` with 192 rows, one per order, with `items`, `total` and `tip` added.**

### Task 3: Describe the data

| Part | What I did | Answer |
|---|---|---|
| 3.1 Types | Set sensible types, with `slot` as an ordered category (Lunch < Dinner < Late night) | see the table in the notebook |
| 3.2 Missing | Counted missing values, plotted them, tested whether missingness depends on other columns | `minutes` missing in 42 orders, `rating` in 24. Delivery time is missing far more often when the **restaurant** delivered (64 % vs 7 %). Rating missingness looks random. |
| 3.3 Distances | Showed both units, converted metres to km (`distance_km`) | The raw column mixes two scales and can't be averaged. |
| 3.4 Delivery time | Mean, median, spread, histogram | Right-skewed: mean 29.8, median 27. The **median** is the better "typical" value. |
| 3.5 By slot | Medians and box plots per slot | No real difference between slots (p = 0.33). |
| 3.6 By city | Counts and shares | Skopje has 51.6 % of the orders. |
| 3.7 Items vs total | Scatter plot, correlation | r = 0.88, about 292 MKD per item, roughly linear but with growing scatter. |
| 3.8 Correlations | Spearman heatmap | Strongest pair: **items and total** (0.87). |
| 3.9 City + slot | Median time per combination, at least 3 timed orders | **Prilep at dinner**, 35 minutes. |

### Task 4: Delivery times (5 % significance, 95 % confidence)
- **4.1** The site claims "28 minutes on average". A one-sample t-test gives p = 0.036, so the claim is **rejected**. The 95 % range for the true average is **28.1 to 31.4** min.
- **4.2** Rain vs clear: Welch's t-test gives p = 0.031, so they **differ**. Rain minus clear is **0.4 to 8.2** min (about 4.3 minutes slower in rain).

### Task 5: Slots and payments
- **5.1** The payment method **depends on the slot** (chi-square p = 0.004). Cash share is 12.5 % at lunch, 27.8 % at dinner, 40.4 % late at night, with 95 % bootstrap ranges in the notebook.
- **5.2** The site claims 35 / 45 / 20 % (lunch / dinner / late night), but the real mix is 29 / 41 / 30 %. The claim is **rejected** (p = 0.003). The late-night share is **23.4 % to 36.5 %**, which excludes the claimed 20 %.

## 5. Statistics glossary

- **Mean / median**: the average vs the middle value. With skewed data the median is more typical.
- **Skewed**: a long tail on one side (a few very slow deliveries here).
- **Hypothesis test**: asks "could the difference be just chance?" The **null hypothesis (H0)** is "nothing is going on".
- **p-value**: how surprising the data would be if H0 were true. If p < 0.05 (the 5 % level), we **reject H0**. If not, we only say there is no evidence, not that H0 is true.
- **Confidence interval (95 %)**: a range of plausible values for the true quantity.
- **t-test**: compares means. **Welch** is the version that doesn't assume equal spreads.
- **Chi-square test**: compares counts in categories (independence, or fit to a stated mix).
- **Bootstrap (percentile)**: resample your data with replacement thousands of times and take the middle 95 % of the resulting estimates as the range.
- **Correlation**: how strongly two numbers move together (-1 to 1). **Spearman** uses ranks, so it is robust to outliers and curves.
- **Multiple testing (Holm)**: testing many things at once makes false alarms likely, so p-values are adjusted upward.

## 6. Things to know before you submit

- **Random numbers:** the notebook fixes the random seed (`rng = default_rng(2026)`), so the bootstrap ranges are the same on every full run. Don't run cells out of order, or the numbers can shift slightly.
- **Keep the outputs visible.** The lab test reads values from the notebook's outputs.
- **Understand it, don't just upload it.** Read the short "Answer" text under each solution cell and make sure you can explain it, since you may be asked about it.
- **Caveat on the times:** 42 orders have no delivery time, mostly restaurant-delivered ones, so the Task 4 answers describe the orders that state a time.
