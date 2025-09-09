# Azure-Cloud-Cost-Analysis
SQL + Power BI project analyzing monthly cloud infrastructure costs, including cleaning, aggregation, and month-over-month trend analysis.

**#SQL Cleaning and Manipulating**
-- Update UsageDate to 'YYYY-MM-DD'
UPDATE cost_analysis
SET UsageDate = DATE(UsageDate)
WHERE UsageDate IS NOT NULL;

-- Quick check on UsageDate
SELECT UsageDate FROM cost_analysis
LIMIT 10;

-- Drop redundant columns
ALTER TABLE cost_analysis DROP COLUMN Cost;
ALTER TABLE cost_analysis DROP COLUMN Currency;

-- Standardize ServiceName
UPDATE cost_analysis
SET ServiceName = TRIM(ServiceName)
WHERE ServiceName IS NOT NULL;

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
