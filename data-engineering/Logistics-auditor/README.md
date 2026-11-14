# Last Mile Logistics Audit: Veridi Logistics

**Candidate:** Jean de Dieu Kwizera | **Track:** Data Engineering

## A. Executive Summary

Late deliveries at Veridi are a regional problem, not a nationwide one. Across 96,470 delivered orders, 93.2% arrived on or before the promised date, but the late rate ranges from 2.8% to 21.4% by state, and the Northeast and Rio de Janeiro receive 22% of orders while accounting for 40% of late deliveries. The delivery estimates are inaccurate in two directions: the typical order arrives 12 days early, yet the same cushion is too thin in the worst-affected states. Lateness is strongly linked to bad reviews, with on-time orders averaging 4.29 stars against 2.27 for late orders, and the drop beginning on the promised date. Late deliveries and low scores also spiked together in November 2017 and February-March 2018, which is most likely the spike the CEO noticed.

## B. Project Links

- **Notebook (Google Colab):** https://colab.research.google.com/drive/14l_2v3qqpQeu2gNnFVa8-zLCmHiEvJXX
- **Dashboard (Looker Studio):** https://datastudio.google.com/reporting/cfd49ada-e159-4331-ade3-783673399adb
- **Presentation:** [ ]
- **Notebook files in this repo:** `last_mile_logistics_audit.ipynb` and `last_mile_logistics_audit.html`

## C. Technical Explanation

### Data cleaning

- **Duplicate reviews.** The reviews table has 99,224 rows for 98,673 orders, so 551 rows are repeat reviews. I kept the most recent review per order before joining, because a plain join would have duplicated orders. Both joins use `validate="one_to_one"`, and an `assert` confirms the master table still has 99,441 rows.
- **Orders that were never delivered.** I classified only orders with status `delivered` and a delivery date (96,470). The other 2,971 are flagged as "Not Delivered" and kept out of the late-rate calculations. This includes 8 orders marked delivered with no delivery date and 6 with a delivery date but a non-delivered status.
- **Delay calculation.** `Days_Difference` is the estimated date minus the actual delivery date, compared as calendar days. The estimate has no time of day, so an order arriving at any hour on the promised date counts as on time. Orders are classified as On Time (0 or more), Late (1 to 5 days late) or Super Late (more than 5 days late).
- **Product categories.** Categories are translated with the dataset's translation file. Two categories missing from that file were mapped manually. Because an order can contain several items, I used the category of the first item to keep one row per order. 2,191 orders with no item record or no category are labelled "unknown".
- **Small samples.** States with fewer than 100 orders (RR, AP, AC) are reported with a caution, and category comparisons use only categories with at least 500 orders.
- **Reproducibility.** The notebook downloads the data with `kagglehub` and uses relative paths only. The raw dataset is not included in this repository.

### Candidate's Choice: recalibrating the delivery promise

For each state, I calculated how many days would need to be added to the delivery estimate so that 95% of orders arrive on or before the promised date. The method takes the 95th percentile of the delay in days for each state with at least 200 orders, then replays past orders with the longer estimates.

**Why it matters to the business:** the analysis shows that a missed promise costs about two review stars, while arriving earlier than promised earns almost nothing. The most direct way to protect review scores is therefore to make promises that hold. This table shows where to act first and by how much: 19 of 23 states would need a longer estimate, with the largest gaps (+12 to +13 days) in the Northeast and Rio de Janeiro, while São Paulo and Minas Gerais need no change. Replaying history with these estimates would have avoided 2,143 broken promises, cutting the national late rate from 6.8% to 4.6%.

**Limits:** this is a replay of past orders, not a forecast. It assumes delivery times stay the same and that longer estimates do not discourage customers from buying. The size of the adjustments also shows that the underlying issue is a minority of very late orders, which longer estimates alone cannot fully fix.

---
