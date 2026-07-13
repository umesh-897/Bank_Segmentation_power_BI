# Bank_Segmentation_power_BI

An end-to-end Power BI dashboard analyzing bank transaction data to uncover customer demographics, transaction behavior, RFM-based segmentation, and a simulated risk/value scoring model.

---

## 📊 Overview

This project transforms a raw bank transactions CSV (~1M rows) into a 4-page interactive Power BI dashboard, covering:

1. **Customer Demographics** — who the customers are
2. **Transaction Behavior** — how and when they transact
3. **Customer Segmentation (RFM)** — who's Loyal, New, Regular, or Lost
4. **Risk & Value Analysis** — a simulated credit risk and customer value model

---

## 🗂️ Dataset

- **Source file:** `bank_transactions.csv`
- **Rows:** ~1,048,567 transactions
- **Unique customers:** ~884,265
- **Date range:** August 1, 2016 – October 21, 2016
- **Grain:** One row per transaction

### Key columns
| Column | Description |
|---|---|
| `TransactionID` | Unique transaction identifier |
| `CustomerID` | Unique customer identifier |
| `CustomerDOB` | Customer date of birth |
| `CustGender` | Customer gender (M/F) |
| `CustLocation` | Customer city |
| `CustAccountBalance` | Account balance at time of transaction |
| `TransactionDate` | Date of transaction |
| `TransactionTime` | Time of transaction (HHMMSS format) |
| `TransactionAmount (INR)` | Transaction amount in ₹ |

---

## 🧹 Data Cleaning & Transformation

Performed in **Power Query**:

- Converted `TransactionDate` and `CustomerDOB` from text to proper Date types (UK locale, DD/MM/YY)
- Split `TransactionTime` (integer HHMMSS) into a usable Time field
- Removed invalid placeholder DOBs (e.g., `1/1/1800`) by nulling out years before 1920
- Standardized blank/null handling across `CustGender` and `CustLocation` (trimmed whitespace, replaced nulls with `"Unknown"`)
- Cleaned a stray `'T'` value found in `CustGender`
- Renamed `TransactionAmount (INR)` → `TransactionAmount_INR` for DAX compatibility

### Derived columns
- `Age` — calculated from cleaned DOB and transaction date
- `AgeBucket` — bucketed age ranges (Under 18, 18-25, 26-35, 36-50, 51-65, 65+, Unknown)
- `TransactionHour`, `TransactionDayOfWeek`, `TransactionMonth`
- `LocationGroup` — Top 20 locations + "Other" bucket (to keep visuals readable given 9,000+ raw location values)

---

## 🧮 Data Model

Two core tables, related via `CustomerID`:

- **`bank_transactions`** — transaction-grain fact table
- **`Customer`** — customer-grain dimension table, built via `Group By` in Power Query:
  - `Frequency` (count of transactions)
  - `Monetary` (sum of transaction amount)
  - `LastTransactionDate`, `FirstTransactionDate`
  - `CustAccountBalance`, `CustGender`, `CustLocation`, `Age`, `AgeBucket`

**Relationship:** `Customer[CustomerID]` (1) → `bank_transactions[CustomerID]` (Many)

---

## 🏷️ RFM Segmentation

Customers are scored on **Recency, Frequency, and Monetary** value using quintile-based scoring (1–5 scale), then classified:

```dax
CustomerSegment =
SWITCH(
    TRUE(),
    Customer[R_Score] >= 4 && Customer[F_Score] >= 4, "Loyal",
    Customer[R_Score] >= 4 && Customer[F_Score] <= 2, "New",
    Customer[R_Score] <= 2 && Customer[F_Score] <= 2, "Lost",
    "Regular"
)
```

| Segment | Meaning |
|---|---|
| **Loyal** | Recently active + frequent transactor |
| **New** | Recently active, low frequency |
| **Lost** | Inactive for a long time, low frequency |
| **Regular** | Everyone else — moderate on one or both dimensions |

---

## 📄 Dashboard Pages

### Page 1 — Customer Demographics
KPIs (Total/Male/Female Customers, Unique Locations), gender distribution, location breakdown, age × gender distribution, customer detail table.

### Page 2 — Transaction Behavior Insights
KPIs (Total Amount, Avg Amount, Max Transaction, Avg Balance), transaction volume over time, day/hour activity heatmap, amount by age group.

### Page 3 — Customer Segmentation (RFM)
KPIs (Loyal/New/Lost counts), segment performance matrix, segment distribution, age distribution by segment, Recency vs. Frequency scatter.

### Page 4 — Risk & Value Analysis *(simulated)*
KPIs (Total Revenue, Avg Simulated Credit Score, High-Risk Count), credit score gauge, revenue decomposition tree, segment/risk ribbon chart, Key Influencers analysis, Top 10 high-value/high-risk customers table.

---

## 📐 Key DAX Measures

```dax
Total Customers = DISTINCTCOUNT(Customer[CustomerID])
Total Transaction Amount = SUM(bank_transactions[TransactionAmount_INR])
Avg Transaction Amount = AVERAGE(bank_transactions[TransactionAmount_INR])
Avg Account Balance = AVERAGE(Customer[CustAccountBalance])
Loyal Customers = CALCULATE(DISTINCTCOUNT(Customer[CustomerID]), Customer[CustomerSegment]="Loyal")
Avg Revenue per Customer = AVERAGE(Customer[Monetary])
```

---

## 💡 Key Business Insights

1. **Customer base skews young and male** — ~73% male, median age ~29, concentrated in metro cities (Mumbai, Delhi, Bangalore).
2. **Typical transactions are small; a few large ones skew the average** — median ₹459 vs. mean ₹1,574.
3. **Wealth is concentrated in a small group** — median balance ~₹17K vs. mean ~₹1.15L.
4. **Peak activity is Sunday evenings around 8 PM**, suggesting consumer/retail-style usage rather than traditional business banking.
5. **Loyal customers are few but valuable** — smallest segment by count, but a disproportionately strong revenue contributor.
6. **"Lost" is the largest segment by count but the smallest by revenue** — a strong win-back opportunity.

---

## 🎨 Design

- **Theme:** Custom Navy & Teal palette (`#0B3954`, `#087E8B`, `#5DA9E9`) for a professional financial look
- **Semantic colors:** Green/Amber/Red reserved exclusively for Risk Level indicators
- Consistent KPI card styling, slicer panel placement, and page header banners across all 4 pages

---

## 🛠️ Tools Used

- **Power BI Desktop** — data modeling, DAX, visualization
- **Power Query (M)** — data cleaning and transformation

---

## Learning Outcomes
Through this project, I strengthened my practical skills in Power BI and data analytics — from cleaning messy real-world data in Power Query (handling nulls, placeholder values, and inconsistent text) to building a proper data model with correctly separated transaction-grain and customer-grain tables. I learned to write and debug DAX calculated columns and measures, implement RFM-based customer segmentation using quintile scoring, and trace data mismatches back to their root cause (like relationship fan-out and blank values being silently excluded from charts). I also practiced responsible analytics — clearly labeling simulated metrics when real data wasn't available — and applied dashboard design principles like consistent theming, appropriate visual selection, and avoiding redundant charts to keep the final report clean and genuinely insightful.

---

## Conclusion
This project took raw, unstructured bank transaction data and transformed it into a complete, interactive Power BI dashboard covering customer demographics, transaction behavior, RFM segmentation, and a simulated risk/value model. The analysis revealed a young, male-skewed, metro-concentrated customer base, transaction amounts driven by a few large outliers rather than typical spend, and a small but valuable Loyal segment alongside a much larger Lost segment worth targeting for re-engagement. Beyond the visuals themselves, the project reinforced that trustworthy dashboards depend on careful data validation at every step, and that acknowledging a dataset's limitations — like its short time span and low repeat-transaction rate — is just as important as the insights it reveals.
