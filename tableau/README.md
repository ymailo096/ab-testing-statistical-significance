# Tableau Dashboard

Interactive dashboard visualizing statistical significance results for all 4 A/B tests.

**Live dashboard:** [A/B Testing — Statistical Significance](https://public.tableau.com/app/profile/yaroslav.mailo/viz/ABTESTINGSTATISTICALSIGNIFICANCE/ABTESTINGSTATISTICALSIGNIFICANCE)

**Screenshot:** see `dashboard_screenshot.png` in this folder (Test 1 view, static snapshot — the live dashboard is interactive and lets you switch between all 4 tests).

## What it shows

- Group 1 / Group 2 conversion rates for each of the 4 metrics
- Absolute change (percentage points) between groups
- Statistical significance (green = significant, red = not significant), based on the two-proportion z-test computed in the Python notebook
- A `Statistical Evidence` table with the exact p-value behind each result

## How significance is determined

Significance values are computed in Python (`statsmodels.stats.proportion.proportions_ztest`) and exported as a CSV — the dashboard doesn't calculate p-values itself, it visualizes results already validated in the notebook. See `notebook/ab_test_significance_analysis.ipynb` for the full methodology.

## Filter by test

Click **Test 1 / Test 2 / Test 3 / Test 4** at the top of the dashboard to switch between experiments.
