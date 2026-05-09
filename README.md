# QuickPay FinTech Operations — Case Study Submission

## Student Information

| Field | Details |
|---|---|
| **Student Name** | [vedant sharma] |
| **Student ID** | [2601008] |
| **Public GitHub Repository Link** | [https://github.com/yourusername/quickpay-fintech](https://github.com/vedant5964sharma-jpg/quickpay-fintech/edit/main/README.md) |

---

## Short run instructions

### Prerequisites
```bash
pip install pandas numpy openpyxl jupyter
```

### Step 1 — Clone the repository
```bash
git clone https://github.com/yourusername/quickpay-fintech.git
cd quickpay-fintech
```

### Step 2 — Run the Python Notebook
```bash
cd 04_python
jupyter notebook fintech_pipeline.ipynb
```
Run all cells top to bottom. This generates all processed output files and summary_metrics.json.

### Step 3 — Review the Spreadsheet
Open `02_spreadsheet/spreadsheet_workbook.xlsx` in Excel or Google Sheets.

### Step 4 — Run SQL Queries
Load `01_data/processed/cleaned_transactions.csv` as `cleaned_transactions` and run `03_sql/analysis_queries.sql`.

### Step 5 — View the Dashboard
Open `05_visualization/dashboard_link.txt` for the Looker Studio public link.

---

## Tools used

| Tool | Purpose |
|---|---|
| Python 3.10+ | Data processing, reconciliation, JSON normalization |
| pandas | DataFrame operations and CSV I/O |
| numpy | Numeric operations and median imputation |
| openpyxl | Excel workbook creation with formatting |
| Jupyter Notebook | Interactive workflow and documentation |
| SQL (SQLite/BigQuery compatible) | Business analysis — 8 queries |
| Excel / Google Sheets | Spreadsheet cleaning and business logic |
| Looker Studio | Dashboard visualization |

---

## Key Findings Summary

| Metric | Value |
|---|---|
| Total GMV (USD) | $116,080.00 |
| Captured GMV (USD) | $82,355.50 |
| Success Rate | 63.33% (19/30) |
| Total High Value Transactions | 7 |
| Total High Risk Transactions | 9 |
| Top Region by GMV | APAC — $82,594.00 |
| Top Merchant by Captured GMV | Beta Stores — $33,431.00 |
| Reconciliation: Missing in Gateway | 2 records (R004, R010) |
| Reconciliation: Missing in Ledger | 1 record (R011) |
| Amount at Risk | $1,490.00 |
| Fraud Alert | U008 — 4 failed/chargeback on 2026-03-05 |
