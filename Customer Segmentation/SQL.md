## Explore the Data
Inspect the data, we have to make sure the columns required for RFM analysis exist 
(Order date, Sales, total orders, and total profit). In this case, we use MS SQL Server and query.


**Firstly, we create a Database**
```sql
Create Database customer_segmentation;
USE customer_segmentation;
```

**Let's find the total number of orders**
```sql
Select count([Order id]) as Order_count
from sales_data;
```
Order_count|
-----------|
9994|

**Total number of customers**

```sql
Select Distinct(count([Customer ID])) as Customer_Count
from customer_data;
```
Customer_Count|
--------------|
793|

**Let's join the tables and view the data using join**

```sql
Select * from customer_data
join sales_data
on 
customer_data.[Customer ID] = sales_data.[Customer ID]
;
```

**What is the most popular segment ?**
```sql
Select Segment,Sum(Sales) as Sales,count(Quantity) as Orders
from  customer_data
join sales_data
on 
customer_data.[Customer ID] = sales_data.[Customer ID]
Group By Segment
Order By Sales DESC
;
```
Segment	|Sales	|Orders|
--------|-------|-------|
Consumer|	1161401.345|	5191|
Corporate|	706146.3668|	3020|
Home Office|	429653.1485	|1783|

*We can see from the results that Consumer segment has the most orders as well as the highest sales*


**Age wise sales analysis**

```sql
Select Distinct(Age),count([Order ID]) as Orders
from  customer_data
join sales_data
on 
customer_data.[Customer ID] = sales_data.[Customer ID]
Group By (Age)
Order By Orders DESC
;
```


**Year wise sales analysis**

```sql
Select Distinct(year([Order date]))as Year,round(sum(sales),2) as Sales, round(sum(profit),2) as Profit
from  customer_data
join sales_data
on 
customer_data.[Customer ID] = sales_data.[Customer ID]
Group By year([Order Date])
Order By Profit DESC
;
```
Year	|Sales	|Profit|
------|--------|------|
2017|	733215.26	|93439.27|
2016|	609205.6	|81795.17|
2015|	470532.51	|61618.6|
2014|	484247.5	|49543.97|

*Year 2017 and 2016 are the most profitable years while 2014 is the least profitable*


**Geographical Sales analysis**
```sql
Select [State], round(sum(sales),2) as Sales, count([Order ID]) as Orders
from  customer_data
join sales_data
on 
customer_data.[Customer ID] = sales_data.[Customer ID]
Group By [State]
Order By Orders DESC
;
```



```sql
Select [Region], round(sum(sales),2) as Sales, count([Order ID]) as Orders
from  customer_data
join sales_data
on 
customer_data.[Customer ID] = sales_data.[Customer ID]
Group By [Region]
Order By Orders DESC
;
```


**Customer with most orders**

```sql
Select [Customer Name],Segment,round(sum(sales),2) as Sales, count([Order ID]) as Orders
from  customer_data
join sales_data
on 
customer_data.[Customer ID] = sales_data.[Customer ID]
Group By [Customer Name],Segment
Order By Orders DESC
;
```

**Average Days required to ship a product**
```sql
Select [City],
AVG(DATEDIFF(DAY,[Order Date],[Ship Date])) as days_for_product_delivery
from  customer_data
join sales_data
on 
customer_data.[Customer ID] = sales_data.[Customer ID]
Group By [City]
Order By days_for_product_delivery ASC;
```

**Most Popular shipping mode among customers**
```sql
Select [Ship Mode],
AVG(DATEDIFF(DAY,[Order Date],[Ship Date])) as avg_days_for_product_delivery
from  customer_data
join sales_data
on 
customer_data.[Customer ID] = sales_data.[Customer ID]
Group By [Ship Mode]
Order By avg_days_for_product_delivery ASC;
```


### RFM Calculation

```sql
WITH RFM_Base 
AS
(
  SELECT b.[Customer Name] AS CustomerName,
    DATEDIFF(DAY, MAX(a.[Order Date]), CONVERT(DATE, GETDATE())) AS Recency_Value,
    COUNT(DISTINCT a.[Order Date]) AS Frequency_Value,
    ROUND(SUM(a.Sales), 2) AS Monetary_Value
  FROM sales_data AS a
  INNER JOIN customer_data AS b ON a.[Customer ID] = b.[Customer ID]
  GROUP BY b.[Customer Name]
)
-- SELECT * FROM RFM_Base
, RFM_Score 
AS
(
  SELECT *,
    NTILE(5) OVER (ORDER BY Recency_Value DESC) as R_Score,
    NTILE(5) OVER (ORDER BY Frequency_Value ASC) as F_Score,
    NTILE(5) OVER (ORDER BY Monetary_Value ASC) as M_Score
  FROM RFM_Base
)
-- SELECT * FROM RFM_Score
, RFM_Final
AS
(
SELECT *,
  CONCAT(R_Score, F_Score, M_Score) as RFM_Overall
  -- , (R_Score + F_Score + M_Score) as RFM_Overall1
  --  CAST(R_Score AS char(1))+CAST(F_Score AS char(1))+CAST(M_Score AS char(1)) as RFM_Overall2
FROM RFM_Score
)
-- SELECT * FROM RFM_Final
SELECT f.*, s.Segment
FROM RFM_Final f
JOIN [segment_score] s ON f.RFM_Overall = s.Scores
;
```
