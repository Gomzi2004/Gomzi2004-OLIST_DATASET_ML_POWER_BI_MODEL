<!--
HOW TO ADD YOUR SCREENSHOTS
Each dashboard section below has a line marked ADD SCREENSHOT.
To add a screenshot in the GitHub web editor: delete the image line
under that mark, then drag your screenshot into the editor at that
place. GitHub will insert its own image link.
If you upload the whole folder instead, keep the screenshots/ folder
with the file names 01-overview.png to 06-precision-and-recall.png
and the image lines below will work without any change.
-->

# Olist Late-Delivery Prediction — Time-Ordered Test

Predict whether a delivered Olist (Brazilian E-Commerce) order will arrive late,
using only information known at purchase time — and test the model on the
newest orders only, the way it would be used in real life.

This is **not** a random train/test split. The orders are sorted by
`purchase_date`, the oldest 80% are the training set, and the newest 20% are
the test set. Test months never appear in training, so the scores below are a
future-month forecast, not an easier random-split score.

---

## 1. About This Project

This project answers one practical question: can we flag, at purchase time,
the orders most likely to arrive late, so they can be checked first?

The model is trained only on information known when the order is placed.
It is tested only on later orders. A flagged order is a shortlist for
review, not proof that it will be late.

What this project contains:

- A complete, beginner-friendly Colab notebook that builds the model
  and the predictions.
- The Power BI dataset built from the model's output.
- A 6-page Power BI dashboard, shown with screenshots in Section 7.

---

## 2. Dataset

Source: Olist Brazilian E-Commerce data, delivered orders only.

| | |
|---|---|
| Orders in the file | 96,470 delivered orders |
| Date range | 2016-09-15 to 2018-08-29 |
| Late orders | 7,826 (8.11%) |
| Revenue (`payment_value`) | R$ 15,421,083 |
| Item value (`total_price`) | R$ 13,220,249 |
| Average payment | R$ 159.85 |

Revenue is `payment_value`, not `total_price`. The two are different:
`total_price` is the item value, `payment_value` is what was paid,
including freight and instalment effects.

The file used by the notebook is `olist_powerbi_orders.csv`, the
`Orders_PowerBI` sheet exported as CSV. The same data is in the Excel
file in `data/`, on the `Orders_PowerBI` sheet.

---

## 3. Time-Ordered Split — Why Not a Random Split

A random split would mix future orders into training and past orders
into testing. Its scores would look better, but they would not show how
the model does on orders it has never seen in time.

This project uses a time-ordered split instead:

| Split | Orders | Dates | Late |
|---|---|---|---|
| Train — oldest 80% | 77,176 | 2016-09-15 to 2018-05-26 | 6,805 (8.82%) |
| Test — newest 20% | 19,294 | 2018-05-26 to 2018-08-29 | 1,021 (5.29%) |

Test months never appear in training. Do not compare these scores
directly with a random-split score. The random 80/20 split on the same
data scored ROC-AUC 0.794 and PR-AUC 0.311, but that test was easier
because its test months overlapped the training months.

---

## 4. Features — Purchase Time Only

Text features: `purchase_month`, `main_category`, `main_seller_state`,
`customer_state`, `payment_type`.

Numeric features: `total_price`, `total_freight`, `payment_value`,
`item_count`, `seller_count`, `estimated_days`.

Never used as features:

- `order_id` — identifier only.
- `purchase_date` — used only for sorting the time split.
- `actual_late` — the target.
- Delivered dates, review scores, or shipping-limit dates.

There is no leakage column in this file.

---

## 5. Models

Three models were trained on the same time-ordered split, for comparison.
The saved predictions come from HistGradientBoosting.

| Model | ROC-AUC | PR-AUC |
|---|---|---|
| Logistic Regression (scaled, balanced) | 0.730 | 0.119 |
| Random Forest (200 trees, balanced) | 0.638 | 0.081 |
| HistGradientBoosting (used for predictions) | 0.717 | 0.100 |

Logistic Regression scored slightly higher on ROC-AUC in this run.
HistGradientBoosting was kept for the saved predictions for consistency
with the earlier runs in this project.

Prediction settings: HistGradientBoosting, threshold 0.10. At threshold
0.50 the model would flag almost nothing, because only 5.29% of the test
set is late. At 0.10 it flags a shortlist to review.

---

## 6. Results

### 6.1 Headline results — test set only

| | |
|---|---|
| ROC-AUC | 0.717 |
| PR-AUC | 0.100 (baseline = test late rate, 0.053) |
| Flagged as late | 4,809 of 19,294 (24.92%) |
| Caught | 504 of 1,021 actually late orders |
| Precision | 0.105 — of the flagged orders, 10.5% were really late |
| Recall | 0.494 — of the really late orders, 49.4% were caught |
| F1 | 0.173 |
| Accuracy | 0.75 |
| Highest predicted probability | 0.558 — no order is flagged with high confidence |

**Read this honestly:** 89.5% of flagged orders were actually on time.
The model catches about half of late orders while flagging a quarter
of all orders.

### 6.2 Confusion matrix — test set

Rows = actual, columns = predicted.

| | Predicted on time | Predicted late | Total |
|---|---|---|---|
| Actually on time | 13,968 | 4,305 | 18,273 |
| Actually late | 517 | 504 | 1,021 |
| Total | 14,485 | 4,809 | 19,294 |

### 6.3 Results by test month

| Test month | Orders | Actually late | Flagged | Caught | Precision | Recall |
|---|---|---|---|---|---|---|
| 2018-05 (stub: test starts 26 May) | 691 | 2 | 3 | 0 | 0.0% | 0.0% |
| 2018-06 | 6,096 | 83 | 503 | 27 | 5.4% | 32.5% |
| 2018-07 | 6,156 | 276 | 1,810 | 151 | 8.3% | 54.7% |
| 2018-08 | 6,351 | 660 | 2,493 | 326 | 13.1% | 49.4% |

Precision rises over time, recall peaks in July, and the model still
misses about half of late orders in August.

### 6.4 Probability bands — does a higher score mean a higher late chance?

Orders are grouped into 0.10-wide probability bands. The label shows
the lower bound: 0.0 means 0.00–0.10, 0.1 means 0.10–0.20, and so on.

| Band | Orders | Late rate |
|---|---|---|
| 0.00–0.10 | 14,485 | 3.57% |
| 0.10–0.20 | 3,761 | 10.40% |
| 0.20–0.30 | 794 | 10.33% |
| 0.30–0.40 | 210 | 10.48% |
| 0.40–0.50 | 42 | 19.05% |
| 0.50–0.60 | 2 | 50.00% |

The late rate is almost flat at about 10.4% from 0.10 to 0.40, so a
higher probability barely means a higher late chance in that range.
The 0.50 band's 50.00% is only 2 orders and is not reliable, and
3,761 of the 4,809 flagged orders sit in the 0.10–0.20 band.

---

## 7. Power BI Dashboard

The dashboard is built from the Excel file in `data/`. It has 6 pages,
in the order below. The screenshots are full Power BI Desktop captures,
taken 28 October 2026, with all slicers cleared, so every page shows
the full unfiltered data.

### 7.1 OVERVIEW — the whole business, all 96,470 delivered orders

<!-- ADD SCREENSHOT 1 HERE: OVERVIEW -->
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/a1183a37-42db-44be-b1c1-cb96b5ac8e29" />


- Headline cards: 96,470 total orders, 7,826 late orders, average payment
  R$ 159.85, revenue R$ 15.42M, late rate 8.11%.
- Payment mix: item value by payment type is dominated by credit card
  (R$ 10.6M), then boleto (R$ 2.3M); debit card, voucher and unknown are
  small. The donut shows the same split on revenue: credit card 79.67%.
- Growth: item value by year rises from almost nothing in 2016 to
  R$ 6.0M in 2017 and R$ 7.2M in 2018 (to 29 August).
- Seller state: São Paulo (SP) is far ahead — 68K orders and R$ 8.5M of
  item value — followed by PR (R$ 1.2M), MG (R$ 1.0M) and RJ (R$ 0.8M).
  Use the bar charts for the state comparison; the map visual plots the
  two-letter state codes, so its pin positions are not the comparison.
- The `main_category` slicer at the top is set to All, so nothing on this
  page is filtered.

### 7.2 Categories and Time — what sells, and how orders grew month by month

<!-- ADD SCREENSHOT 2 HERE: Categories and Time -->
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/678a7099-e700-4182-951d-bef41310f09f" />


- Average item value by category (top 7): `fixed_telephony` is highest at
  R$ 10.3K, then `housewares` R$ 6.7K and `art` R$ 6.5K. A high average
  does not mean high volume — most categories average R$ 4K–6K.
- Orders by payment type: credit card 74.3K orders (77.01%), boleto
  19.19K (19.89%), voucher 1.5K (1.55%), debit card and unknown are tiny.
- Item value by month (`purchase_month_yyyy_mm`, 23 months, Sep 2016 to
  Aug 2018, in time order): a steady climb from near zero to about
  R$ 0.99M in Dec 2017, then roughly R$ 0.73M–0.98M a month through 2018.
- Orders and late rate by month: order volume grows to 6K–7K a month in
  2018. Ignore the late-rate line's first point, 100% in Sep 2016, when
  there were almost no orders; after that the monthly late rate sits
  roughly between 0% and 21%, peaking around Mar–Apr 2018.
- The `main_category` slicer is set to All.

### 7.3 ML MEASURES — the model, on the full test set only

<!-- ADD SCREENSHOT 3 HERE: ML MEASURES -->
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/bf78780d-41a6-4620-8b80-ed7e8f5b83b5" />


- The note on the page states the setup in full: HistGradientBoosting,
  threshold 0.10, train oldest 80% to 26 May 2018 (77,176 orders), test
  newest 20% 26 May to 29 Aug 2018 (19,294 orders), ROC-AUC 0.717,
  PR-AUC 0.100 (baseline 0.053), flagged 4,809 (24.92%), caught 504 of
  1,021 late (recall 49.4%), precision 10.5%. Flag means check first,
  not proof of late.
- The cards show the same measures rounded to K (for example Test Orders
  19.294K, Test Predicted Late 5K, True Negatives 14K). Read exact values
  from the note and the confusion matrix, not from the rounded cards:
  Accuracy 0.75, F1 0.17, Test Actual Late 1,021, Test Late Rate 5.29%,
  True Positives 504, False Negatives 517, False Positives 4,305,
  True Negatives 13,968.
- Confusion matrix: 13,968 / 4,305 / 18,273; 517 / 504 / 1,021;
  totals 14,485 / 4,809 / 19,294. The donut beside it shows the same
  split: 14.49K (75.08%) not flagged, 4.81K (24.92%) flagged.
- Where the flags fall: most flagged test orders are in `health_beauty`
  (651), then `bed_bath_table` (453) and `watches_gifts` (369); by state,
  SP (1,397) is far ahead, then RJ (731) and BA (431). These are counts
  of flagged orders, so large categories and states lead — they are not
  late rates.
- By test month, actual late vs flagged: May 2 / 3, June 83 / 503,
  July 276 / 1,810, August 660 / 2,493. Flagging rises with the late
  count, but in every month most flagged orders were on time.
- This page has no slicer and must stay on the full test set. Do not
  sync a slicer from another page to it.

### 7.4 Flagged Orders — the review shortlist, highest probability first

<!-- ADD SCREENSHOT 4 HERE: Flagged Orders -->
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/497d7f51-c94c-4d27-9b8a-2f68b4cfb821" />


- The table lists flagged test orders only (`predicted_late = 1`),
  sorted by `predicted_probability` descending, so the riskiest-looking
  orders come first. Each row shows the order, its category, customer
  state, payment type, predicted probability and whether it was
  actually late.
- The highest probability in the whole test set is 0.56 (a
  `health_beauty` order, BA, credit card, actually late). The next rows
  are 0.51, 0.50 and 0.49 — most of them were actually on time, which is
  why a flag is a check-first shortlist, not a prediction of certainty.
- Slicers on the right, all cleared in the screenshots: test month
  (`purchase_month_yyyy_mm`: 2018-05 to 2018-08), `main_category`
  (All) and `customer_state` (All). These slicers belong to this page
  only and are not synced to the other pages, so using them here never
  changes the ML MEASURES page.
- Use this page to work through the shortlist month by month or
  category by category, then clear the slicers to return to all
  4,809 flagged orders.

### 7.5 Probability Bands — does a higher score mean a higher late chance?

<!-- ADD SCREENSHOT 5 HERE: Probability Bands -->
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/0e5f1439-0503-4039-9c72-677f63d30b9b" />


- The chart groups test orders into 0.10-wide probability bands and
  shows the order count (bars) and the late rate (line) in each band.
  See the band table in Section 6.4 for the exact numbers.
- The late rate is almost flat at about 10.4% from 0.10 to 0.40.
  The 0.50 band's 50.00% is only 2 orders and is not reliable.

### 7.6 Precision and Recall — how the model did in each test month

<!-- ADD SCREENSHOT 6 HERE: Precision and Recall -->
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/1a71060d-01af-4993-953a-1ea58ca5c6b2" />

- Precision = of the orders flagged, how many were really late.
  Recall = of the orders that were really late, how many were caught.
- The line chart in the screenshot shows June to August only. May is
  stated in the note above the chart and in the table in Section 6.3,
  because May is only a stub: the test set starts 26 May, with 691
  orders, 2 late, 3 flagged and none caught, so both measures are
  0.0% for May.
- Reading: precision rises over time (5.4% → 8.3% → 13.1%), recall
  peaks in July (54.7%), and the model still misses about half of
  late orders in August.

---

## 8. How to Run the Model

1. Open `POWER_BI_OLIST.ipynb` in Google Colab.
2. Upload `olist_powerbi_orders.csv` to the Colab session.
3. Run all cells. The outputs in the notebook are the run quoted above.
4. Download `prediction.csv` from the last cell.

The notebook is simple, straight-line code with no function definitions.
Change only the one `FILE_PATH` line at the top if your file is in a
different place.

In Power BI, join `prediction` to `Orders_PowerBI` on `order_id`:
one-to-one (1:1), cross-filter direction Both, Active.

---

## 9. Files in This Repo

| File | What it is |
|---|---|
| `POWER_BI_OLIST.ipynb` | The complete Colab notebook: 2 cells, simple straight-line code. Run it in Google Colab. |
| `data/olist_powerbi_dataset_time_split_colab.xlsx` | Power BI dataset. Sheets: `Orders_PowerBI` (96,470 delivered orders), `prediction` (19,294 test orders only, this notebook's output), `Model_Metrics`, `README`. |
| `screenshots/` | The 6 dashboard screenshots used in Section 7, in page order. |
| `requirements.txt` | Python packages for running the notebook outside Colab: pandas, scikit-learn. |
| `DESCRIPTION.txt` | The short GitHub repository description and suggested topics. |

---

## 10. Limitations

- Precision is low: only 10.5% of flagged orders were really late.
  The model is a review shortlist, not a decision on its own.
- The highest predicted probability is only 0.558. No order is
  flagged with high confidence.
- May 2018 in the test set is only a 6-day stub. Do not read a trend
  from May alone.
- The probability score does not rank late risk well between 0.10
  and 0.40. See Section 6.4.
- The dashboard cards round large counts to K. Always read exact
  values from the note and the confusion matrix on the ML MEASURES page.

---


