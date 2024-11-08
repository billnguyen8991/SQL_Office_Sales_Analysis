**<h1 align="center"> Discount Strategy Analysis </h1>**

**Purpose**

This project aims to explore Discount Strategy of the company on a several range of office products to understand employees performance and products, sales trend of different products on different level of discount so that the organisation can improve and optimise their strategy.

**Task**

**Question 1:**
Write a SQL query to calculate the total sales of furniture products, grouped by each quarter of the year, and order the results chronologically?
````sql
WITH YearCTE AS (
    SELECT
        ORDER_DATE,
        PRODUCT_ID,
        sales,
        p.NAME, 
        CASE 
            WHEN DATEPART(Quarter, ORDER_DATE) = 1 THEN 'Q1-' + CAST(YEAR(ORDER_Date) AS VARCHAR(4))
            WHEN DATEPART(Quarter, ORDER_DATE) = 2 THEN 'Q2-' + CAST(YEAR(ORDER_Date) AS VARCHAR(4))
            WHEN DATEPART(Quarter, ORDER_DATE) = 3 THEN 'Q3-' + CAST(YEAR(ORDER_Date) AS VARCHAR(4))
            WHEN DATEPART(Quarter, ORDER_DATE) = 4 THEN 'Q4-' + CAST(YEAR(ORDER_Date) AS VARCHAR(4))
        END AS Quarter_Year
    FROM 
        Orders o
    JOIN 
        Product p ON o.PRODUCT_ID = p.ID
)

SELECT 
    Quarter_Year,
    ROUND(SUM(sales), 2) AS total_sales
FROM 
    YearCTE
WHERE 
    NAME = 'Furniture'
GROUP BY 
    Quarter_Year
ORDER BY 
    RIGHT(Quarter_Year, 4), RIGHT(LEFT(Quarter_Year, 2), 1);
````

**Result:**
| Quarter_Year | Total_Sales |
|--------------|-------------|
| Q1-2014      | 22656.14    |
| Q2-2014      | 28063.75    |
| Q3-2014      | 41957.88    |
| Q4-2014      | 64515.09    |
| Q1-2015      | 27374.10    |
| Q2-2015      | 27564.83    |
| Q3-2015      | 49586.04    |
| Q4-2015      | 65993.28    |
| Q1-2016      | 24349.39    |
| Q2-2016      | 41402.50    |
| Q3-2016      | 52814.63    |
| Q4-2016      | 80334.92    |
| Q1-2017      | 23723.81    |
| Q2-2017      | 45032.10    |
| Q3-2017      | 56283.10    |
| Q4-2017      | 90348.25    |



**Question 2:** Write a query to analyze the impact of different discount levels on sales performance across product categories, 
specifically looking at the number of orders and total profit generated for each discount classification?

Discount level condition:\
No Discount = 0\
0 < Low Discount < 0.2\
0.2 < Medium Discount < 0.5\
High Discount > 0.5 


````sql
WITH DiscountCTE AS (
    SELECT 
        DISCOUNT,
		CATEGORY,
		PROFIT,
        CASE 
            WHEN DISCOUNT = 0 THEN 'No Discount'
            WHEN DISCOUNT <= 0.2 THEN 'Low Discount'
            WHEN DISCOUNT <= 0.5 THEN 'Medium Discount'
            ELSE 'High Discount'
        END AS Discount_Level
    FROM 
        Orders o
	join PRODUCT p on o.PRODUCT_ID = p.ID
)

select CATEGORY,
		Discount_Level,
		COUNT(*) as Total_Orders,
		Round(SUM(PROFIT),0) as Total_Profit
from DiscountCTE
group by CATEGORY, Discount_Level
order by CATEGORY, Discount_Level
````
**Result:**
| Category     | Discount_Level   | Total_Orders | Total_Profit |
|--------------|------------------|--------------|--------------|
| Accessories  | Medium Discount   | 304          | 6647         |
| Accessories  | No Discount       | 471          | 35289        |
| Appliances   | High Discount     | 67           | -8630        |
| Appliances   | Low Discount      | 16           | 1086         |
| Appliances   | Medium Discount   | 112          | 2498         |
| Appliances   | No Discount       | 271          | 23184        |
...
...
...
| Machines     | Low Discount      | 2            | 832          |
| Machines     | Medium Discount   | 61           | -5006        |
| Phones       | No Discount       | 311          | 34365        |
| Storage      | No Discount       | 530          | 25528        |
| Supplies     | Medium Discount   | 73           | -2907        |
| Supplies     | No Discount       | 117          | 1718         |
| Tables       | Medium Discount   | 247          | -30582       |
| Tables       | No Discount       | 72           | 13276        |


**Question 3:**
Write a query to determine the top-performing product categories within each customer segment based on sales and profit, 
focusing specifically on those categories that rank within the top two for profitability?
````sql
WITH RankedData AS (
    SELECT 
        c.SEGMENT,
        p.CATEGORY,
        SUM(o.SALES) AS Total_Sales,
        SUM(o.PROFIT) AS Total_Profit,
        Dense_RANK() OVER(PARTITION BY c.SEGMENT ORDER BY SUM(o.SALES) DESC) AS Sales_Rank,
        DENSE_RANK() OVER(PARTITION BY c.SEGMENT ORDER BY SUM(o.PROFIT) DESC) AS Profit_Rank
    FROM 
        Orders o
    JOIN 
        Customers c ON c.ID = o.CUSTOMER_ID
    JOIN 
        Product p ON p.ID = o.PRODUCT_ID
    GROUP BY 
        c.SEGMENT, p.CATEGORY
)
SELECT 
    SEGMENT,
    CATEGORY,
    Sales_Rank,
    Profit_Rank
FROM 
    RankedData
WHERE 
    Profit_Rank <= 2
ORDER BY 
    SEGMENT, CATEGORY;
````

**Result:**


| Segment        | Category    | Sales_Rank | Profit_Rank |
|----------------|-------------|------------|-------------|
| Consumer       | Copiers     | 8          | 1           |
| Consumer       | Phones      | 2          | 2           |
| Corporate      | Accessories  | 7          | 2           |
| Corporate      | Copiers     | 8          | 1           |
| Home Office    | Copiers     | 7          | 1           |
| Home Office    | Phones      | 1          | 2           |


**Question 4:**
Write a query to create a report that displays each employee's performance across different product categories, 
showing not only the total profit per category but also what percentage of their total profit each category represents, 
with the results ordered by the percentage in descending order for each employee?

````sql
SELECT 
    e.ID_EMPLOYEE,
    p.CATEGORY,
	Round(sum(o.SALES),2) as Total_Sales,
    ROUND(SUM(o.PROFIT), 2) AS Round_Total_Profit,
    ROUND((SUM(o.PROFIT) / SUM(SUM(o.PROFIT)) OVER (PARTITION BY e.ID_Employee)) * 100, 2) AS Profit_Percentage
FROM 
    Employees e
JOIN 
    Orders o ON e.ID_EMPLOYEE = o.ID_EMPLOYEE
JOIN 
    Product p ON p.ID = o.PRODUCT_ID
GROUP BY 
    e.ID_EMPLOYEE, p.CATEGORY
ORDER BY 
    e.ID_EMPLOYEE ASC, Round_Total_Profit DESC;
````
**Result:**

| ID_EMPLOYEE | CATEGORY      | TOTAL_SALES | TOTAL_PROFIT | PROFIT_PERCENTAGE |
|-------------|---------------|-------------|--------------|--------------------|
| 1           | Copiers       | 9699.77     | 4360.9       | 19.25              |
| 1           | Phones        | 21480.05    | 3262.29      | 14.4               |
| 1           | Binders       | 10505.17    | 3212.77      | 14.18              |
| 1           | Accessories   | 11505.6     | 2929.34      | 12.93              |
| 1           | Paper         | 3594.96     | 1643.98      | 7.26               |
| 1           | Envelopes     | 747.34      | 339.7        | 1.5                |
...
...
...
| 9           | Copiers       | 27699.62    | 8912.88      | 21.55              |
| 9           | Accessories   | 27200.33    | 6937.45      | 16.77              |
| 9           | Chairs        | 61514.25    | 6338.22      | 15.32              |
| 9           | Phones        | 45079.29    | 5671.05      | 13.71              |
| 9           | Paper         | 11427.22    | 4996.81      | 12.08              |
| 9           | Storage       | 34866.65    | 3321.65      | 8.03               |
| 9           | Binders       | 20157.74    | 2463.12      | 5.95               |


**Question 5:**
Write a query to develop a user-defined function in SQL Server to calculate the profitability ratio for each product 
category an employee has sold, and then apply this function to generate a report that ranks each employee's product 
categories by their profitability ratio?

````sql
CREATE FUNCTION dbo.CalculateProfitabilityRatio (@EmployeeID INT)
RETURNS TABLE
AS
RETURN
(
    SELECT 
        p.CATEGORY,
		ROUND(SUM(o.SALES),2) as Total_Sales,
        ROUND(SUM(o.PROFIT), 2) AS Total_Profit,
        ROUND(SUM(o.PROFIT) / NULLIF(SUM(o.SALES), 0), 2) AS Profitability_Ratio
    FROM 
        Orders o
    INNER JOIN 
        Product p ON p.ID = o.PRODUCT_ID
    WHERE 
        o.ID_EMPLOYEE = @EmployeeID
    GROUP BY 
        p.CATEGORY
);

SELECT 
    e.ID_EMPLOYEE,
    c.CATEGORY,
	COALESCE(pr.Total_Sales, 0) as Total_Sales,
    COALESCE(pr.Total_Profit, 0) AS Total_Profit,
    COALESCE(pr.Profitability_Ratio, 0) AS Profitability_Ratio
FROM 
    Employees e
CROSS JOIN 
    (SELECT DISTINCT CATEGORY FROM Product) c -- All product categories
LEFT JOIN 
    (
        SELECT 
            o.ID_EMPLOYEE,
            p.CATEGORY,
			ROUND(SUM(o.SALES),2) as Total_Sales,
            ROUND(SUM(o.PROFIT), 2) AS Total_Profit,
            ROUND(SUM(o.PROFIT) / NULLIF(SUM(o.SALES), 0), 2) AS Profitability_Ratio
        FROM 
            Orders o
        INNER JOIN 
            Product p ON p.ID = o.PRODUCT_ID
        GROUP BY 
            o.ID_EMPLOYEE, p.CATEGORY
    ) pr ON e.ID_EMPLOYEE = pr.ID_EMPLOYEE AND c.CATEGORY = pr.CATEGORY
ORDER BY 
    e.ID_EMPLOYEE, Profitability_Ratio DESC;
````

**Result:**

| Employee ID | Category    | Total Sales | Total Profit | Profitability Ratio |
|-------------|-------------|-------------|--------------|---------------------|
| 1           | Labels      | 1289.38     | 617.49       | 0.48                |
| 1           | Paper       | 3594.96     | 1643.98      | 0.46                |
| 1           | Envelopes   | 747.34      | 339.7        | 0.45                |
| 1           | Copiers     | 9699.77     | 4360.9       | 0.45                |
| 1           | Fasteners   | 164.55      | 62.42        | 0.38                |
| 1           | Binders     | 10505.17    | 3212.77      | 0.31                |
| 1           | Accessories | 11505.6     | 2929.34      | 0.25                |
...
| 8           | Chairs       | 60808.64    | 4188.40      | 0.07                |
| 8           | Binders      | 42225.52    | 1016.88      | 0.02                |
| 8           | Supplies     | 8501.11     | 5.03         | 0                   |
| 8           | Bookcases    | 19329.13    | -1545.43     | -0.08               |
| 9           | Chairs       | 61514.25    | 6338.22      | 0.10                |
| 9           | Machines     | 31060.70    | 147.76       | 0                   |
| 9           | Tables       | 34634.10    | -2155.57     | -0.06               |
| 9           | Bookcases    | 14477.13    | -963.15      | -0.07               |
| 9           | Supplies     | 5040.55     | -695.37      | -0.14               |


**Question 6:** Write a query to create a report that displays the performance of orders across different discount levels.
````sql
WITH AggregatedDiscounts AS (
    SELECT 
        CASE 
            WHEN DISCOUNT = 0 THEN 'No Discount'
            WHEN DISCOUNT <= 0.2 THEN 'Low Discount'
            WHEN DISCOUNT <= 0.5 THEN 'Medium Discount'
            ELSE 'High Discount'
        END AS Discount_Level,
        COUNT(*) AS Total_Orders,
        SUM(SALES) AS Total_Sales,
        SUM(PROFIT) AS Total_Profit,
        AVG(DISCOUNT) AS Avg_Discount
    FROM 
        Orders o
    JOIN 
        PRODUCT p ON o.PRODUCT_ID = p.ID
    GROUP BY 
        CASE 
            WHEN DISCOUNT = 0 THEN 'No Discount'
            WHEN DISCOUNT <= 0.2 THEN 'Low Discount'
            WHEN DISCOUNT <= 0.5 THEN 'Medium Discount'
            ELSE 'High Discount'
        END
)

SELECT 
    Discount_Level,
    Total_Orders,
    ROUND(Total_Sales, 0) AS Total_Sales,
    ROUND(Total_Profit, 0) AS Total_Profit,
    ROUND(Total_Sales / NULLIF(Total_Orders, 0), 2) AS Sales_Per_Order,
    ROUND(Total_Profit / NULLIF(Total_Orders, 0), 2) AS Profit_Per_Order,
    ROUND(Avg_Discount, 2) AS Avg_Discount_Percentage
FROM 
    AggregatedDiscounts
ORDER BY 
    CASE 
        WHEN Discount_Level = 'High Discount' THEN 1
        WHEN Discount_Level = 'Medium Discount' THEN 2
        WHEN Discount_Level = 'Low Discount' THEN 3
        WHEN Discount_Level = 'No Discount' THEN 4
    END;  -- Order by Discount Level
````
**Result:**
| Discount Level  | Total Orders | Total Sales | Total Profit | Sales Per Order | Profit Per Order | Avg Discount Percentage |
|-----------------|--------------|-------------|--------------|-----------------|------------------|-------------------------|
| High Discount   | 856          | 64,229      | -76,559      | 75.03           | -89.44           | 0.72                    |
| Medium Discount | 4,194        | 1,063,136   | 31,940       | 253.49          | 7.62             | 0.22                    |
| Low Discount    | 146          | 81,928      | 10,448       | 561.15          | 71.56            | 0.12                    |
| No Discount     | 4,798        | 1,087,908   | 320,988      | 226.74          | 66.9             | 0                       |

**Question 7:** Write a query to analyse customer pattern based on their order history.

````sql
with RFM_CTE as (
select CUSTOMER_ID,
		DATEDIFF(day, MAX(ORDER_DATE),'2017-12-31') as recency,
		DATEDIFF(day, MIN(order_date),'2017-12-31')/COUNT(*) as frequency,
		SUM(sales)/COUNT(*) as monetary
from Orders
group by CUSTOMER_ID)

select *,
	NTILE(5) over (order by recency) as n_tile_recency,
	PERCENTILE_DISC(0.2) within group (order by recency) over () as percent_20_recency,
	PERCENTILE_DISC(0.4) within group (order by recency) over () as percent_40_recency,
	PERCENTILE_DISC(0.6) within group (order by recency) over () as percent_60_recency,
	PERCENTILE_DISC(0.8) within group (order by recency) over () as percent_80_recency,

	ntile(5) over (order by frequency) as n_tile_frequency,
	percentile_disc(0.2) within group (order by frequency) over () as percent_20_frequency,
	percentile_disc(0.4) within group (order by frequency) over () as percent_40_frequency,
	percentile_disc(0.6) within group (order by frequency) over () as percent_60_frequency,
	PERCENTILE_DISC(0.8) within group (order by frequency) over () as percent_80_frequency,

	NTILE(5) over (order by monetary) as n_tile_monetary,
	PERCENTILE_DISC(0.2) within group (order by monetary) over () as percent_20_monetary,
	PERCENTILE_DISC(0.4) within group (order by monetary) over () as percent_40_monetary,
	PERCENTILE_DISC(0.6) within group (order by monetary) over () as percent_60_monetary,
	PERCENTILE_DISC(0.8) within group (order by monetary) over () as percent_80_monetary
into RFM_RawData
from RFM_CTE



WITH RFM_CalData AS (
    SELECT 
        Customer_ID,
        (CASE 
            WHEN recency <= (SELECT MAX(percent_20_recency) FROM RFM_RawData) THEN 1
            WHEN recency <= (SELECT MAX(percent_40_recency) FROM RFM_RawData) THEN 2
            WHEN recency <= (SELECT MAX(percent_60_recency) FROM RFM_RawData) THEN 3 
            WHEN recency <= (SELECT MAX(percent_80_recency) FROM RFM_RawData) THEN 4 
            ELSE 5 
        END) AS rb,
        
        (CASE 
            WHEN frequency <= (SELECT MAX(percent_20_frequency) FROM RFM_RawData) THEN 1
            WHEN frequency <= (SELECT MAX(percent_40_frequency) FROM RFM_RawData) THEN 2
            WHEN frequency <= (SELECT MAX(percent_60_frequency) FROM RFM_RawData) THEN 3
            WHEN frequency <= (SELECT MAX(percent_80_frequency) FROM RFM_RawData) THEN 4
            ELSE 5 
        END) AS fb, 
        
        (CASE 
            WHEN monetary <= (SELECT MAX(percent_20_monetary) FROM RFM_RawData) THEN 1
            WHEN monetary <= (SELECT MAX(percent_40_monetary) FROM RFM_RawData) THEN 2
            WHEN monetary <= (SELECT MAX(percent_60_monetary) FROM RFM_RawData) THEN 3
            WHEN monetary <= (SELECT MAX(percent_80_monetary) FROM RFM_RawData) THEN 4
            ELSE 5 
        END) AS mb
    FROM RFM_RawData
)

SELECT 
    rb * 100 + fb * 10 + mb AS rfm,  -- Calculate the RFM score
    COUNT(Customer_ID) AS Customer_Count  -- Count of customers for each RFM score
FROM RFM_CalData
GROUP BY rb * 100 + fb * 10 + mb  -- Group by the calculated RFM score
ORDER BY rfm;  -- Order by the RFM score
````
**Results:**
| ID   | Value |
|------|-------|
| 111  | 6     |
| 112  | 14    |
| 113  | 9     |
| 114  | 13    |
| 115  | 8     |
| 121  | 6     |
...
| 544  | 10    |
| 545  | 5     |
| 551  | 29    |
| 552  | 12    |
| 553  | 9     |
| 554  | 6     |
| 555  | 13    |
