# Bank Customer Segmentation: RFM Analysis, K-Means Clustering and Power BI Dashboard

Segmenting **882,600 bank customers** from about 1.05 million transactions shows that the **top 10% of customers generate 63% of transaction value**, and that **84% of customers made only one transaction**. The biggest opportunity is converting high-value one-time customers into repeat customers.

<img width="772" height="490" alt="Screenshot 2026-11-12 080152" src="https://github.com/user-attachments/assets/cafb2d1f-c57d-4ca4-b0e4-0a3d5581b03a" />


## Business problem

Understanding customer behavior helps a bank retain customers and grow customer lifetime value. This project turns raw transaction records into a small number of actionable customer segments and shows where the value sits.

**Questions answered**
- Which customer segments exist, and how do they differ in recency, frequency, spend and balance?
- How concentrated is transaction value across customers?
- Which segment should be prioritized, and why?

## Dataset

- Public Kaggle bank transactions dataset (`bank_transactions.csv`): (https://www.kaggle.com/datasets/shivamb/bank-customer-segmentation)
- Columns used: `CustomerID`, `TransactionID`, `TransactionDate`, `TransactionAmount (INR)`, `CustAccountBalance`
- All monetary values are in **INR**.
- After cleaning: **1,046,198 valid transactions** across **882,600 customers**.

## Methodology

1. **Cleaning.** Dropped rows missing customer ID, date, transaction amount or account balance, and parsed transaction dates.
2. **RFM feature engineering** per customer:
   - *Recency*: days since the last transaction, measured from the day after the latest date in the data
   - *Frequency*: number of transactions
   - *MonetaryValue*: total transaction amount (INR)
   - *MeanBalance*: average account balance (INR)
3. **Preprocessing.** `log10(x + 0.1)` to reduce heavy right skew, then standard scaling.
4. **Choosing k.** Elbow method (WCSS) and silhouette score (sampled 10,000 customers) for k = 2 to 7. **k = 4** was selected.
5. **Clustering.** K-Means with `k=4`, `n_init=10`, `random_state=42`.
6. **Profiling.** Clusters were named from their measured profile (lowest recency, highest frequency, then transaction value and balance), not from cluster numbers.

## Customer segments

| Segment | Customers | % of customers | % of value | Avg recency (days) | Avg frequency | Value per customer (INR) | Median balance (INR) |
|---|---|---|---|---|---|---|---|
| High-Value One-Timers | 420,422 | 47.6% | 62.8% | 143 | 1.00 | 2,458 | 39,066 |
| Loyal Repeaters | 130,826 | 14.8% | 26.8% | 100 | 2.14 | 3,376 | 29,542 |
| Recently Active | 42,632 | 4.8% | 5.3% | 1.5 | 1.34 | 2,027 | 18,928 |
| Low-Value One-Timers | 288,720 | 32.7% | 5.1% | 143 | 1.00 | 293 | 4,163 |

Total transaction value: **about ₹1.65 billion**.

## Key findings

- **Value is concentrated.** The top 10% of customers hold 63% of total transaction value and the top 20% hold about 77%.
- **The base is mostly one-time customers.** 84% made a single transaction in the data window.
- **The biggest pool is High-Value One-Timers.** They are 48% of customers and 63% of value, with the highest median balance.
- **Loyal Repeaters are small but valuable.** 15% of customers generate 27% of value, with the highest value per customer.
- **Low-Value One-Timers** are a third of customers but only 5% of value.

## Recommended actions

These are hypotheses based on the segment profiles, not model output.

| Segment | Priority | Suggested action |
|---|---|---|
| High-Value One-Timers | 1 | Convert to repeat: personal follow-up, second-transaction incentive, cross-sell savings or investment products |
| Loyal Repeaters | 2 | Retain and grow: loyalty rewards, premium or bundled products, early-warning check if activity slows |
| Recently Active | 3 | Nurture early: onboarding journey and a nudge toward the next transaction |
| Low-Value One-Timers | 4 | Keep cost low: automated, low-touch campaigns only |

## Power BI dashboard

**Visuals**
- KPI cards: total customers, median balance, single-transaction %, top 10% value share
- Pareto chart (line and clustered column): transaction value by customer decile with cumulative share
- Clustered bar: share of customers vs share of transaction value per segment
- Segment summary table
- Slicers: segment (tile), recency bucket and balance tier (dropdown)

**Data model and DAX highlights**
- `Persona Fixed`: calculated column that maps each cluster to its segment name
- `MonetaryDecile`: created in Power Query (sort by value, add rank, divide into 10 equal groups)
- Measures: `Customer Share %` and `Value Share %` (using `ALLSELECTED`), `Value Share of Decile`, `Cumulative Value Share`, `Single-Transaction %`, `Top 10% Value Share`, `Median Balance`

## Limitations

- **No churn label.** Segments are descriptive. "One-Timers" and "Recently Active" are not churn predictions.
- **Two segments differ mainly in value.** High-Value and Low-Value One-Timers are almost all single-transaction customers with similar recency; they separate mostly on transaction value and balance. "Recently Active" is separated almost entirely by recency.
- **Recency pattern.** No customer has a last transaction 8 to 30 days before the snapshot date, which looks like a property of the data's date distribution. Recency-based conclusions should be read with care.
- **Cluster stability** across random seeds and other values of k was not tested.
- Recommended actions have not been tested with campaigns or experiments.

## Repository structure

```
notebooks/   Bank_Segmentation_Analysis.ipynb
dashboard/   Power BI file (.pbix)
images/      dashboard screenshots
data/        sample_rfm.csv (1,000-row sample; full data not included)
```

## How to reproduce

1. Download `bank_transactions.csv` from the Kaggle link above and place it next to the notebook.
2. Install dependencies: `pip install pandas numpy matplotlib seaborn scikit-learn`
3. Run the notebook. It writes `rfm_segmented_customers.csv`.
4. Open the `.pbix` in Power BI Desktop. To refresh with your own copy, update the file path under Transform data > Data source settings.

## Tech stack

Python (pandas, NumPy, scikit-learn, matplotlib, seaborn), Power BI (Power Query, DAX), Git

## Author

**Satriyo Akbar Maulana**, Information Systems (Business Intelligence), Universitas Mercu Buana
[LinkedIn](https://linkedin.com/in/satriyo-akbar-maulana-b23894220/) · [GitHub](https://github.com/FixMyCodeNow)
