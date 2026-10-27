# Olist Late-Delivery Prediction — Time-Ordered Test

Predict whether a delivered Olist (Brazilian E-Commerce) order will arrive late,
using only information known at purchase time — and test the model on the
newest orders only, the way it would be used in real life.

This is **not** a random train/test split. The orders are sorted by
`purchase_date`, the oldest 80% are the training set, and the newest 20% are
the test set. Test months never appear in training, so the scores below are a
future-month forecast, not an easier random-split score.

## Result at a glance

| | |
|---|---|
| Model used for predictions | HistGradientBoosting (threshold 0.10) |
| Train | Oldest 80%: 77,176 orders, 2016-09-15 to 2018-05-26, 6,805 late (8.82%) |
| Test | Newest 20%: 19,294 orders, 2018-05-26 to 2018-08-29, 1,021 late (5.29%) |
| ROC-AUC | 0.717 |
| PR-AUC | 0.100 (baseline = test late rate, 0.053) |
| Flagged as late | 4,809 of 19,294 (24.92%) |
| Caught | 504 of 1,021 actually late orders |
| Precision | 0.105 — of the flagged orders, 10.5% were really late |
| Recall | 0.494 — of the really late orders, 49.4% were caught |
| F1 | 0.173 |

Confusion matrix, test set (rows = actual, columns = predicted):

| | Predicted on time | Predicted late |
|---|---|---|
| Actually on time | 13,968 | 4,305 |
| Actually late | 517 | 504 |

**Read this honestly:** a flagged order is a shortlist for review, not proof
it will be late. 89.5% of flagged orders were actually on time. The highest
predicted probability in the whole test set is only 0.558 — no order is
flagged with high confidence. The model catches about half of late orders
while flagging a quarter of all orders.

## Results by test month

| Test month | Orders | Actually late | Flagged | Caught | Precision | Recall |
|---|---|---|---|---|---|---|
| 2018-05 (stub: test starts 26 May) | 691 | 2 | 3 | 0 | 0.0% | 0.0% |
| 2018-06 | 6,096 | 83 | 503 | 27 | 5.4% | 32.5% |
| 2018-07 | 6,156 | 276 | 1,810 | 151 | 8.3% | 54.7% |
| 2018-08 | 6,351 | 660 | 2,493 | 326 | 13.1% | 49.4% |

Precision rises over time, recall peaks in July, and the model still misses
about half of late orders in August.

Probability bands (0.10 bins, test set): the late rate is almost flat at
about 10.4% from 0.10 to 0.40 (3,761 / 794 / 210 orders), then 19.05% at
0.40–0.50 (42 orders) and 50.00% at 0.50–0.60 on only 2 orders, which is not
reliable. A higher predicted probability barely means a higher late chance
until 0.40+.

## Reference models in the same notebook

These were trained on the same time-ordered split, for comparison only.
The saved predictions come from HistGradientBoosting.

| Model | ROC-AUC | PR-AUC |
|---|---|---|
| Logistic Regression (scaled, balanced) | 0.730 | 0.119 |
| Random Forest (200 trees, balanced) | 0.638 | 0.081 |
| HistGradientBoosting (used for predictions) | 0.717 | 0.100 |

Logistic Regression scored slightly higher on ROC-AUC in this run;
HistGradientBoosting was kept for the saved predictions for consistency with
the earlier runs in this project.

## Features — purchase time only

Text: `purchase_month`, `main_category`, `main_seller_state`,
`customer_state`, `payment_type`.
Numeric: `total_price`, `total_freight`, `payment_value`, `item_count`,
`seller_count`, `estimated_days`.

Never used as features: `order_id` (identifier only), `purchase_date`
(sorting only), `actual_late` (the target), delivered dates, review scores,
or shipping-limit dates. There is no leakage column in this file.

## Files

| File | What it is |
|---|---|
| `POWER_BI_OLIST.ipynb` | The complete Colab notebook: 2 cells, simple straight-line code, no function definitions. Run it in Google Colab. |
| `data/olist_powerbi_dataset_time_split_colab.xlsx` | Power BI dataset. Sheets: `Orders_PowerBI` (96,470 delivered orders), `prediction` (19,294 test orders only, this notebook's output), `Model_Metrics`, `README`. |
| `requirements.txt` | Python packages for running the notebook outside Colab. |

The notebook reads `olist_powerbi_orders.csv` — the `Orders_PowerBI` sheet
exported as CSV (96,470 rows, 7,826 late, 8.11%, `payment_value`
R$ 15,421,083). In Colab, upload that CSV to the session first, or change
the one `FILE_PATH` line at the top of the notebook. The notebook saves
`prediction.csv` (19,294 rows, the same 13 columns as the `prediction` sheet
in the Excel file). Join `prediction` to `Orders_PowerBI` in Power BI on
`order_id`: one-to-one (1:1), cross-filter direction Both, Active.

## How to reproduce

1. Open `POWER_BI_OLIST.ipynb` in Google Colab.
2. Upload `olist_powerbi_orders.csv` to the Colab session.
3. Run all cells. The outputs in the notebook are the run quoted above.
4. Download `prediction.csv` from the last cell.

## What is NOT in this repo

- No random-split prediction file. The random 80/20 split (ROC-AUC 0.794,
  PR-AUC 0.311, 4,644 flagged, 1,018 caught) is **not** comparable to the
  time-ordered result above — its test months overlapped the training
  months, so it was an easier test. Do not load both prediction files in
  one Power BI model.
- No old 32,150-row prediction file.
- No raw Olist CSVs. The dataset is the Olist Brazilian E-Commerce data;
  the `Orders_PowerBI` sheet here is the delivered-orders file prepared
  for this project.
- No Power BI `.pbix` file yet. The Excel file in `data/` is the dataset
  the dashboard is built from.

## Checked

The uploaded notebook and Excel file were re-checked row by row before
posting: 4 sheets only, no old prediction sheet; 96,470 Orders rows and
19,294 prediction rows with no duplicate `order_id`s; every prediction
`order_id` exists in Orders and its `actual_late` agrees with Orders;
`predicted_late` matches `predicted_probability >= 0.10` on every row;
test months are 2018-05 to 2018-08 only; confusion, precision, recall,
and per-month numbers above all recompute exactly from the prediction sheet.
