# Spreadsheet Answers

---

## Cleaning Steps

1. **Trimmed whitespace** — removed all leading/trailing spaces from merchant_name, status, risk_score, and gateway_region columns.
2. **Standardized merchant names** — collapsed all name variants (case, spacing, double spaces) to the canonical form using a lookup table.
3. **Standardized transaction dates** — all dates converted to `YYYY-MM-DD` ISO format.
4. **Standardized status values** — extracted core intent from 10 raw variants into 3 clean values: `captured`, `failed`, `chargeback`.
5. **Standardized risk scores** — stripped `score:` and `risk-` prefixes, extracted numeric values. One null (T011) imputed with column median = 62.0.
6. **Standardized gateway_region** — trimmed and uppercased. 8 null values filled from `merchant_master.csv` default_region via VLOOKUP.
7. **Currency conversion** — joined with `exchange_rates.csv` on `(transaction_date, currency)` to get date-specific USD rates. Applied: `amount_usd = raw_amount × usd_rate`.
8. **Merchant enrichment** — joined on merchant_name to bring in `merchant_id`, `merchant_category` from merchant_master.csv.
9. **Computed business flags** — applied high_value_flag and high_risk_flag rules per business specification.

---

## Standardization Rules

### Merchant Name Mapping
| Raw Value(s) | Standardized |
|---|---|
| alpha mart, ALPHA MART, Alpha  Mart, Alpha Mart | Alpha Mart |
| BETA STORES, Beta  Stores, beta stores, Beta Stores | Beta Stores |
| City Pharma | City Pharma |
| Eco Home | Eco Home |
| DELTA TRAVELS, Delta Travels, delta travels | Delta Travels |

### Status Mapping
| Raw Pattern | Standardized |
|---|---|
| captured, Captured, CAPTURED, captured (trailing space) | captured |
| failed e05 timeout, FAILED e05 TIMEOUT, Failed E05 Timeout | failed |
| chargeback, chargeback (with spaces) | chargeback |

### Risk Score Cleaning
| Raw Pattern | Cleaned |
|---|---|
| `score:62` | 62 |
| `risk-83` | 83 |
| `55`, `71` (plain numeric) | 55, 71 |
| NaN (T011) | 62.0 (median imputation) |

### Gateway Region Mapping
| Raw Value | Standardized |
|---|---|
| APAC, apac, APAC (with spaces) | APAC |
| EU, eu, EU (with spaces) | EU |
| US, us | US |
| NaN (8 rows) | Filled from merchant_master.default_region |

---

## Lookup and Enrichment Logic

### Exchange Rate Conversion (date-specific)
Joined transactions on `(transaction_date, currency)` with exchange_rates.csv:

| Date | Currency | Rate | Example Transaction |
|---|---|---|---|
| 2026-03-01 | INR | 0.0119 | T001: 420,000 × 0.0119 = **$4,998.00** |
| 2026-03-01 | EUR | 1.0800 | T023: 5,200 × 1.0800 = **$5,616.00** |
| 2026-03-01 | USD | 1.0000 | T027: 7,200 × 1.0000 = **$7,200.00** |
| 2026-03-02 | INR | 0.0120 | T004: 160,000 × 0.0120 = **$1,920.00** |

Rates varied by date (INR ranged 0.0118–0.0121, EUR ranged 1.07–1.09).

### Merchant Master Enrichment (VLOOKUP by merchant_name)
Added `merchant_id` and `merchant_category` from merchant_master.csv.
Used `default_region` to fill 8 null gateway_region values.

### Flag Rules Applied
**high_value_flag = 1 when:**
- APAC AND amount_usd > 5,000
- EU AND amount_usd > 6,000
- US AND amount_usd > 7,000
- Otherwise 0

**high_risk_flag = 1 when:**
- risk_score >= 70
- OR status = 'chargeback'
- Otherwise 0

---

## Final Answers

| Metric | Value |
|---|---|
| **Total raw rows** | 30 |
| **Total cleaned rows** | 30 |
| **Invalid or missing rows handled** | 0 rows dropped. 1 null risk_score imputed (T011 → 62.0). 8 null gateway_region values filled from merchant_master. |
| **Top region by GMV** | **APAC** — $82,594.00 USD |
| **Number of high value transactions** | **7** (T003, T007, T010, T014, T020, T024, T027) |
| **Number of high risk transactions** | **9** (T003, T007, T010, T013, T014, T017, T018, T024, T029) |
| **Top merchant by captured GMV** | **Beta Stores** — $33,431.00 USD |

### High Value Transactions Detail
| Transaction | Merchant | Region | Amount USD | Threshold |
|---|---|---|---|---|
| T003 | Beta Stores | APAC | $6,069.00 | >$5,000 APAC |
| T007 | Alpha Mart | APAC | $5,400.00 | >$5,000 APAC |
| T010 | Beta Stores | APAC | $7,381.00 | >$5,000 APAC |
| T014 | Beta Stores | APAC | $5,640.00 | >$5,000 APAC |
| T020 | Alpha Mart | APAC | $6,136.00 | >$5,000 APAC |
| T024 | Eco Home | EU | $6,649.00 | >$6,000 EU |
| T027 | Delta Travels | US | $7,200.00 | >$7,000 US |

### High Risk Transactions Detail
| Transaction | Merchant | Status | Risk Score | Reason |
|---|---|---|---|---|
| T003 | Beta Stores | captured | 71 | risk_score ≥ 70 |
| T007 | Alpha Mart | chargeback | 83 | chargeback + risk ≥ 70 |
| T010 | Beta Stores | captured | 77 | risk_score ≥ 70 |
| T013 | Beta Stores | captured | 73 | risk_score ≥ 70 |
| T014 | Beta Stores | captured | 73 | risk_score ≥ 70 |
| T017 | Beta Stores | failed | 72 | risk_score ≥ 70 |
| T018 | Beta Stores | chargeback | 86 | chargeback + risk ≥ 70 |
| T024 | Eco Home | chargeback | 65 | chargeback |
| T029 | Delta Travels | chargeback | 58 | chargeback |

---

## Formula Samples

### amount_usd (Excel — lookup date+currency combined key)
```excel
=IFERROR(
  VLOOKUP(A2&C2, exchange_lookup_table, 3, 0) * D2,
  D2
)
```
Where exchange_lookup_table has a helper column combining rate_date & currency.

### Risk score cleaning (handles score:xx, risk-xx, plain numeric)
```excel
=IF(ISNUMBER(SEARCH("score:",G2)),
    VALUE(MID(G2, 7, 10)),
    IF(ISNUMBER(SEARCH("risk-",G2)),
        VALUE(MID(G2, 6, 10)),
        IFERROR(VALUE(TRIM(G2)), MEDIAN(risk_score_range))
    )
)
```

### high_value_flag
```excel
=IF(OR(
    AND(H2="APAC", K2>5000),
    AND(H2="EU",   K2>6000),
    AND(H2="US",   K2>7000)
), 1, 0)
```

### high_risk_flag
```excel
=IF(OR(L2>=70, ISNUMBER(SEARCH("chargeback", F2))), 1, 0)
```

### Merchant name standardization
```excel
=IFERROR(
  VLOOKUP(LOWER(TRIM(SUBSTITUTE(C2,"  "," "))), merchant_name_map, 2, 0),
  PROPER(TRIM(C2))
)
```
