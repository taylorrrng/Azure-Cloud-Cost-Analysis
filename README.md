# Azure-Cloud-Cost-Analysis
SQL + Power BI project analyzing monthly cloud infrastructure costs, including cleaning, aggregation, and month-over-month trend analysis.

**Intro**
(will fill out later)

**Goals**
(will fill out later)

**SQL Questions** (questions + code + tables)

1. Convert UsageDate to a real date
```sql
-- Update UsageDate to 'YYYY-MM-DD'
UPDATE cost_analysis
SET UsageDate = DATE(UsageDate)
WHERE UsageDate IS NOT NULL;

-- Quick check on UsageDate
SELECT UsageDate FROM cost_analysis
LIMIT 10;
```
2. Drop unnecesary columns
```
-- Drop redundant columns
ALTER TABLE cost_analysis DROP COLUMN Cost;
ALTER TABLE cost_analysis DROP COLUMN Currency;
```
[View cleaned data](cleaned_data.csv)

Sample:

| UsageDate  | ServiceName             | CostUSD       |
|------------|-------------------------|---------------|
| 2023-05-01 | Automation              | 0.001         |
| 2023-05-01 | Azure DNS               | 0.50034793    |
| 2023-05-01 | Bandwidth               | 127.7707188536|
| 2023-05-01 | Storage                 | 134.5045017768|
| 2023-05-01 | Virtual Machines        | 13.987309323  |
| 2023-05-01 | Virtual Machines License| 23.733881824  |
| 2023-05-01 | Virtual Network         | 22.8286641447 |
| 2023-06-01 | Azure DNS               | 0.4842213677  |
| 2023-06-01 | Bandwidth               | 121.8232207427|
| 2023-06-01 | Logic Apps              | 0.0031706504  |

3. Ensure ServiceName has values
```
-- Standardize ServiceName
UPDATE cost_analysis
SET ServiceName = TRIM(ServiceName)
WHERE ServiceName IS NOT NULL;
```
4. Calculate monthly costs

```
-- Aggregate costs per month for all services
DROP VIEW IF EXISTS view_monthly_cost;
CREATE VIEW IF NOT EXISTS view_monthly_cost AS
	SELECT
		DATE(strftime('%Y-%m-01', UsageDate)) AS Month,
		ROUND(SUM(CostUSD), 2) AS TotalCostUSD
	FROM cost_analysis
GROUP BY Month
ORDER BY Month;

-- Quick check on view
SELECT * FROM view_monthly_cost;
SELECT * FROM cost_analysis;
```
[View monthly cost data](manipulated_data.csv)

Sample:
| Month      | TotalCostUSD |
|------------|--------------|
| 2023-05-01 | 323.33       |
| 2023-06-01 | 311.12       |
| 2023-07-01 | 324.91       |
| 2023-08-01 | 355.54       |
| 2023-09-01 | 361.54       |
| 2023-10-01 | 366.61       |
| 2023-11-01 | 375.60       |
| 2023-12-01 | 312.31       |
| 2024-01-01 | 312.43       |
| 2024-02-01 | 308.93       |

**Tableau Dashboard**

[View dashboard pdf](Tableau Dashboard.pdf)
