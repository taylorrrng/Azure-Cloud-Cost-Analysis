# Introduction
This project analyzes monthly cloud infrastructure costs using SQL for data cleaning and aggregation, Tableau for visualization, and a regression-based forecast (with the assistance of ChatGPT) to project costs for 2025–2026.

# Goals
- Clean and transform raw cost data in SQL for structured analysis

- Aggregate monthly totals to evaluate cost trends

- Build Tableau dashboards to practice different visualization types

- Identify top cost drivers, low-usage services, and peak spending months

- Apply an ML regression model (via ChatGPT) to forecast future costs

# SQL Questions (Questions + Codes + Tables)

**1. Convert UsageDate to a real date**
```sql
-- Update UsageDate to 'YYYY-MM-DD'
UPDATE cost_analysis
SET UsageDate = DATE(UsageDate)
WHERE UsageDate IS NOT NULL;

-- Quick check on UsageDate
SELECT UsageDate FROM cost_analysis
LIMIT 10;
```
**2. Drop unnecesary columns**
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

**3. Ensure ServiceName has values**
```
-- Standardize ServiceName
UPDATE cost_analysis
SET ServiceName = TRIM(ServiceName)
WHERE ServiceName IS NOT NULL;
```
**4. Calculate monthly costs**

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

# Tableau Dashboard

[View dashboard pdf](Tableau_Dashboard.pdf)

<img width="4526" height="3486" alt="dashboard1" src="https://github.com/user-attachments/assets/796bce38-65e9-4ec7-823c-f7280176e19f" />

# Azure Cost Analysis

**Why this exists:** I built a small dashboard with multiple visuals that all look at the same data. The goal was to practice different chart types and see how each one helps me “see” the story a little differently.

**What’s on the page**

- Bar chart: Cost by Service Name split by year (fast comparisons).

- Stacked area: Monthly trend (how total spend and each service move over time).

- Treemap: Share of total cost by service (who’s biggest).

- Bubble chart: Relative size of services by year

- Heatmap: Service × Year with color by cost (quick “where is it heavy?” scan).

**Big takeaways**

***1) 2023 vs 2024 (by service)***

- Most services cost more in 2023 than in 2024.

- The top spenders in 2023 were Storage (~$1,154) and Bandwidth (~$1,022).
In 2024, those dropped to ~$556 and ~$528, respectively.

- That’s roughly a 52% drop for Storage and 48% drop for Bandwidth year-over-year
(not 94–108% — impossible to drop by more than 100% 😅).

TL;DR: Storage carries the biggest dollars across the whole period.

***2) Low / no usage categories***

Several services show little to no spend across both years:

- Automation

- Azure DNS

- Logic Apps

- Network Watcher

***3) Time pattern***

April 2024 is the peak month in this slice, landing just under $450 total.
The stacked area makes that jump easy to spot.

How each visual helps (in one line)

- Bar chart: “Who’s biggest this year vs last year?”

- Stacked area: “When did costs spike or dip month to month?”

- Treemap: “Who takes the largest share of the pie overall?”

- Bubble chart: “How big is each service compared to the others in scale?”

- Heatmap: “Is 2023 or 2024 heavier by service at a glance?”

# ML Regression Prediction

A forecast visualization was generated with the assistance of ChatGPT, using a regression model on the Azure cost data:

- Light pink line with dots → actual monthly costs (2023–2024)

- Hot pink line → regression-based predictions for 2025 and 2026

- Dashed gray line → cutoff point where forecasting begins

**1. Key Insights**

- Costs are forecasted to rise steadily into 2025–2026, reaching ~$520 by mid-2026.

- Compared to the 2023–2024 average (~$350–$400), this suggests a 30–40% increase over two years.

- The regression highlights a clear upward linear trend despite short-term dips (e.g., early 2024).

**2. Methodology**

- Model used: Linear Regression (Ordinary Least Squares), applied via ChatGPT

- Inputs (X): Month index (May 2023 = 1, June 2023 = 2, …)

- Output (y): Monthly total cost (USD)

- Training process: A best-fit line (Cost = a + b * MonthIndex) was fitted to minimize error

- Forecasting: The regression line was extended by 24 months to cover 2025–2026

**3. Limitations**

- Assumes a steady linear growth trend

- Does not account for seasonality or sudden spikes/drops

- Serves as a baseline directional forecast rather than precise month-level predictions
