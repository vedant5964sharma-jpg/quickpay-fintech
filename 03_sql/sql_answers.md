# SQL Answers

---

## Q1
### Query
```sql
SELECT status, COUNT(*) AS transaction_count,
       ROUND(COUNT(*) * 100.0 / SUM(COUNT(*)) OVER (), 2) AS pct_of_total
FROM cleaned_transactions
GROUP BY status ORDER BY transaction_count DESC;
```
### Result Summary
| status | transaction_count | pct_of_total |
|---|---|---|
| captured | 19 | 63.33% |
| failed | 7 | 23.33% |
| chargeback | 4 | 13.33% |

19 out of 30 transactions were captured (63.33%). 4 chargebacks (13.33%) and 7 failures (23.33%) represent significant revenue risk.

---

## Q2
### Query
```sql
SELECT merchant_id, merchant_name,
       ROUND(SUM(amount_usd), 2) AS captured_gmv_usd,
       COUNT(*) AS captured_transactions
FROM cleaned_transactions
WHERE status = 'captured'
GROUP BY merchant_id, merchant_name ORDER BY captured_gmv_usd DESC;
```
### Result Summary
| merchant_name | captured_gmv_usd | captured_transactions |
|---|---|---|
| Beta Stores | 33,431.00 | 9 |
| Alpha Mart | 29,984.50 | 9 |
| Delta Travels | 10,300.00 | 2 |
| City Pharma | 8,640.00 | 2 |
| Eco Home | 0.00 | 0 |

Beta Stores leads with $33,431 captured GMV. Eco Home had 0 captured transactions (all failed or chargeback).

---

## Q3
### Query
```sql
SELECT merchant_id, merchant_name, merchant_category, gateway_region,
       ROUND(SUM(amount_usd), 2) AS captured_gmv_usd,
       COUNT(*) AS captured_transactions
FROM cleaned_transactions WHERE status = 'captured'
GROUP BY merchant_id, merchant_name, merchant_category, gateway_region
ORDER BY captured_gmv_usd DESC LIMIT 10;
```
### Result Summary
Top 4 merchants with captured transactions (dataset has 5 merchants total):
1. **Beta Stores** — $33,431.00 (Electronics, APAC)
2. **Alpha Mart** — $29,984.50 (Grocery, APAC)
3. **Delta Travels** — $10,300.00 (Travel, US)
4. **City Pharma** — $8,640.00 (Healthcare, EU)

APAC dominates the top two positions. Combined APAC captured GMV = $63,415.50 out of total $82,355.50.

---

## Q4
### Query
```sql
SELECT transaction_date,
       ROUND(SUM(amount_usd), 2) AS total_gmv_usd,
       ROUND(SUM(CASE WHEN status='captured' THEN amount_usd ELSE 0 END),2) AS captured_gmv_usd,
       COUNT(*) AS total_transactions,
       SUM(CASE WHEN status='captured' THEN 1 ELSE 0 END) AS successful_transactions,
       ROUND(SUM(CASE WHEN status='captured' THEN 1 ELSE 0 END)*100.0/COUNT(*),2) AS success_rate_pct
FROM cleaned_transactions GROUP BY transaction_date ORDER BY transaction_date;
```
### Result Summary
| date | total_gmv_usd | captured_gmv_usd | total_txns | success_count | success_rate_pct |
|---|---|---|---|---|---|
| 2026-03-01 | 26,382.00 | 26,382.00 | 5 | 5 | 100.00% |
| 2026-03-02 | 25,049.00 | 11,080.00 | 6 | 3 | 50.00% |
| 2026-03-03 | 18,391.00 | 16,031.50 | 5 | 4 | 80.00% |
| 2026-03-04 | 16,420.00 | 13,920.00 | 5 | 4 | 80.00% |
| 2026-03-05 | 19,232.00 | 6,136.00 | 6 | 1 | 16.67% |
| 2026-03-06 | 10,606.00 | 8,806.00 | 3 | 2 | 66.67% |

March 1st had perfect 100% success rate. March 5th crashed to 16.67% — caused by U008's 4 failed/chargeback transactions across multiple merchants.

---

## Q5
### Query
```sql
SELECT merchant_id, merchant_name, COUNT(*) AS total_transactions,
       SUM(CASE WHEN status='chargeback' THEN 1 ELSE 0 END) AS chargeback_count,
       ROUND(SUM(CASE WHEN status='chargeback' THEN 1 ELSE 0 END)*100.0/COUNT(*),2) AS chargeback_ratio_pct,
       ROUND(SUM(CASE WHEN status='chargeback' THEN amount_usd ELSE 0 END),2) AS chargeback_amount_usd
FROM cleaned_transactions GROUP BY merchant_id, merchant_name
HAVING chargeback_ratio_pct > 1 ORDER BY chargeback_ratio_pct DESC;
```
### Result Summary
| merchant_name | total_transactions | chargeback_count | chargeback_ratio_pct | chargeback_amount_usd |
|---|---|---|---|---|
| Eco Home | 2 | 1 | 50.00% | 6,649.00 |
| Delta Travels | 4 | 1 | 25.00% | 2,500.00 |
| Alpha Mart | 11 | 1 | 9.09% | 5,400.00 |
| Beta Stores | 11 | 1 | 9.09% | 1,711.00 |

All 4 merchants exceed the 1% threshold. Eco Home is critical at 50% — it had only 2 transactions total, one being a chargeback. City Pharma is the only clean merchant (0% chargeback).

---

## Q6
### Query
```sql
SELECT gateway_region, COUNT(*) AS transaction_count,
       ROUND(AVG(risk_score), 2) AS avg_risk_score,
       ROUND(SUM(amount_usd), 2) AS total_gmv_usd
FROM cleaned_transactions GROUP BY gateway_region
HAVING AVG(risk_score) > 50 AND COUNT(*) > 20
ORDER BY avg_risk_score DESC;
```
### Result Summary
| gateway_region | transaction_count | avg_risk_score | total_gmv_usd |
|---|---|---|---|
| APAC | 22 | 65.27 | 82,594.00 |

Only APAC qualifies — it has 22 transactions (> 20) and avg risk score 65.27 (> 50).
EU: avg 47.25, count 4 — fails both conditions.
US: avg 48.75, count 4 — fails both conditions.
APAC's high risk combined with its dominant $82,594 GMV makes it the top monitoring priority.

---

## Q7
### Query
```sql
SELECT user_id, transaction_date, COUNT(*) AS failed_chargeback_count
FROM cleaned_transactions WHERE status IN ('failed', 'chargeback')
GROUP BY user_id, transaction_date
HAVING COUNT(*) >= 3 ORDER BY failed_chargeback_count DESC, transaction_date;
```
### Result Summary
| user_id | transaction_date | failed_chargeback_count |
|---|---|---|
| U008 | 2026-03-05 | 4 |

U008 triggered 4 failed/chargeback transactions on March 5th (T016, T017, T018, T019) across Beta Stores and Alpha Mart — a clear fraud signal requiring immediate account review.

---

## Q8
### Query
```sql
SELECT merchant_id, merchant_name,
       COUNT(*) AS chargeback_count,
       COUNT(DISTINCT user_id) AS unique_affected_users,
       ROUND(SUM(amount_usd), 2) AS total_chargeback_amount_usd,
       ROUND(AVG(amount_usd), 2) AS avg_chargeback_amount_usd
FROM cleaned_transactions WHERE status = 'chargeback'
GROUP BY merchant_id, merchant_name ORDER BY total_chargeback_amount_usd DESC;
```
### Result Summary
| merchant_name | chargeback_count | unique_affected_users | total_chargeback_usd | avg_chargeback_usd |
|---|---|---|---|---|
| Eco Home | 1 | 1 | 6,649.00 | 6,649.00 |
| Alpha Mart | 1 | 1 | 5,400.00 | 5,400.00 |
| Delta Travels | 1 | 1 | 2,500.00 | 2,500.00 |
| Beta Stores | 1 | 1 | 1,711.00 | 1,711.00 |

Total chargeback exposure: **$16,260.00** across 4 merchants, 4 unique users.
Eco Home has the highest single chargeback at $6,649. Each merchant has exactly 1 chargeback — no repeat chargeback offenders, but all merchants need monitoring.
