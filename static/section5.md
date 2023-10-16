## ⭐ <span style="color: #0063B2FF;">SQL SERVER - SECTION 5</span>⭐

## 🔗 <span style="color: #0063B2FF;">TEMPORARY TABLES</span>

<span style="color: #0063B2FF;"></span>
<span style="color: #ee6c4d;"></span>

$${\color{red}Welcome \space \color{lightblue}To \space \color{orange}Stackoverflow}$$
A temporary SQL table, also known as a temp table, is a table that is created and used within the context of a specific session or transaction in a database management system. It is designed to store temporary data that is needed for a short duration and does not require a permanent storage solution.
<br>
<br>
Temporary tables in SQL provide a convenient way to break down complex problems into smaller, more manageable steps. They allow for the separation of data processing stages, which can improve performance, enhance code readability, and simplify query logic.
<br>
<br>
Temporary tables can be used in various database systems such as MySQL, PostgreSQL, Oracle, SQL Server, and others, although the syntax and features may vary slightly between implementations.

<details>
<summary><h3 style="color: #0063B2FF;">1. TEMPORARY TABLES</h3>

<span style="color: #ee6c4d;">Temporary tables</span> are created on-the-fly and are typically used to perform complex calculations, store intermediate results, or manipulate subsets of data during the execution of a query or a series of queries.

These temporary tables have a specific scope and lifespan associated with them. They are only accessible within the session or transaction that created them and are automatically dropped or deleted when the session or transaction ends or when explicitly dropped by the user.

This temporary nature of the tables makes them suitable for managing data that is transient and does not need to persist beyond the immediate task at hand.

</summary>

```sql
SELECT -- temporary table Sales
    OrderDate,
    OrderMonth = DATEFROMPARTS(YEAR(OrderDate),MONTH(OrderDate),1),
    TotalDue,
    OrderRank = ROW_NUMBER() OVER(PARTITION BY DATEFROMPARTS(YEAR(OrderDate),MONTH(OrderDate),1) ORDER BY TotalDue DESC)

    INTO #Sales
    FROM AdventureWorks2019.Sales.SalesOrderHeader

SELECT -- temporary table Top10Sales
	OrderMonth,
	Top10Total = SUM(TotalDue)

    INTO #Top10Sales
    FROM #Sales
    WHERE OrderRank <= 10
    GROUP BY OrderMonth

SELECT -- query on the temporary tables
	A.OrderMonth,
	A.Top10Total,
	PrevTop10Total = B.Top10Total

    FROM #Top10Sales A
        LEFT JOIN #Top10Sales B
            ON A.OrderMonth = DATEADD(MONTH,1,B.OrderMonth)

    ORDER BY 1

DROP TABLE #Sales -- drop temporary table
DROP TABLE #Top10Sales -- drop temporary table
```

![Screenshot](/assets/temp_table.png?raw=true "temporary table")

</details>

<details>
<summary><h3 style="color: #0063B2FF;">2. CREATE & INSERT ON TEMPORARY TABLES</h3>

You can use temporary tables to optimize complex or resource-intensive queries. By breaking down a complex query into multiple steps using temporary tables, we can improve query performance by reducing the amount of data processed at each stage or by precomputing intermediate results.

Temporary tables allow us to store and reuse intermediate query results, avoiding redundant computations. Here's an example:

</summary>

```sql
-- temporary table #Sales
CREATE TABLE #Sales (
    OrderDate DATE,
    OrderMonth DATE,
    TotalDue MONEY,
    OrderRank INT)
INSERT INTO #Sales (
    OrderDate,
    OrderMonth,
    TotalDue,
    OrderRank)
SELECT
    OrderDate,
    OrderMonth = DATEFROMPARTS(YEAR(OrderDate),MONTH(OrderDate),1),
    TotalDue,
    OrderRank = ROW_NUMBER() OVER(PARTITION BY DATEFROMPARTS(YEAR(OrderDate),MONTH(OrderDate),1) ORDER BY TotalDue DESC)

    FROM AdventureWorks2019.Sales.SalesOrderHeader

-- temporary table #SalesMinusTop10
CREATE TABLE #SalesMinusTop10 (
    OrderMonth DATE,
    TotalSales MONEY)
INSERT INTO #SalesMinusTop10 (
    OrderMonth,
    TotalSales)
SELECT
    OrderMonth,
    TotalSales = SUM(TotalDue)
    FROM #Sales
    WHERE OrderRank > 10
    GROUP BY OrderMonth

-- temporary table #SalesMinusTop10
CREATE TABLE #SalesMinusTop10 (
    OrderMonth DATE,
    TotalSales MONEY)
INSERT INTO #SalesMinusTop10 (
    OrderMonth,
    TotalSales)
SELECT
    OrderMonth,
    TotalSales = SUM(TotalDue)
    FROM #Sales
    WHERE OrderRank > 10
    GROUP BY OrderMonth

-- temporary table #Purchases
CREATE TABLE #Purchases (
    OrderDate DATE,
    OrderMonth DATE,
    TotalDue MONEY,
    OrderRank INT)
INSERT INTO #Purchases (
    OrderDate,
    OrderMonth,
    TotalDue,
    OrderRank)
SELECT
    OrderDate,
    OrderMonth = DATEFROMPARTS(YEAR(OrderDate),MONTH(OrderDate),1),
    TotalDue,
    OrderRank = ROW_NUMBER() OVER(PARTITION BY DATEFROMPARTS(YEAR(OrderDate),MONTH(OrderDate),1) ORDER BY TotalDue DESC)

    FROM AdventureWorks2019.Purchasing.PurchaseOrderHeader

-- create temporary table #PurchaseMinusTop10
CREATE TABLE #PurchaseMinusTop10 (
    OrderMonth DATE,
    TotalPurchases MONEY)
INSERT INTO #PurchaseMinusTop10 (
    OrderMonth,
    TotalPurchases)
SELECT
    OrderMonth,
    TotalPurchases = SUM(TotalDue)
    FROM #Purchases
    WHERE OrderRank > 10
    GROUP BY OrderMonth

-- use temporary tables
SELECT
    A.OrderMonth,
    A.TotalSales,
    B.TotalPurchases

    FROM #SalesMinusTop10 A
        JOIN #PurchaseMinusTop10 B
            ON A.OrderMonth = B.OrderMonth

    ORDER BY 1

-- DROP all temporary tables
DROP TABLE #Sales
DROP TABLE #SalesMinusTop10
DROP TABLE #Purchases
DROP TABLE #PurchaseMinusTop10

```

![Screenshot](/assets/tp_create_insert.png?raw=true "temporary tables create & insert")

</details>

<details>
<summary><h3 style="color: #0063B2FF;">3. TRUNCATE()</h3>

Removes all rows from a table or specified partitions of a table, without logging the individual row deletions. TRUNCATE TABLE is similar to the DELETE statement with no WHERE clause; however, TRUNCATE TABLE is faster and uses fewer system and transaction log resources.

</summary>

```sql
-- temp table for #Orders
CREATE TABLE #Orders (
     OrderDate DATE
    ,OrderMonth DATE
    ,TotalDue MONEY
    ,OrderRank INT)
INSERT INTO #Orders (
     OrderDate
    ,OrderMonth
    ,TotalDue
    ,OrderRank)
SELECT
     OrderDate
    ,OrderMonth = DATEFROMPARTS(YEAR(OrderDate),MONTH(OrderDate),1)
    ,TotalDue
    ,OrderRank = ROW_NUMBER() OVER(PARTITION BY DATEFROMPARTS(YEAR(OrderDate),MONTH(OrderDate),1) ORDER BY TotalDue DESC)

    FROM AdventureWorks2019.Sales.SalesOrderHeader

-- temp table for #OrdersMinusTop10
CREATE TABLE #OrdersMinusTop10 (
     OrderMonth DATE
    ,OrderType VARCHAR(50)
    ,TotalDue MONEY)
INSERT INTO #OrdersMinusTop10 (
     OrderMonth
    ,OrderType
    ,TotalDue)
SELECT
    OrderMonth,
    OrderType = 'Sales',
    TotalDue = SUM(TotalDue)

    FROM #Orders
    WHERE OrderRank > 10
    GROUP BY OrderMonth

-- Empty out #Orders table
TRUNCATE TABLE #Orders

-- temp table 2 for #Orders
INSERT INTO #Orders (
     OrderDate
    ,OrderMonth
    ,TotalDue
    ,OrderRank)
SELECT
     OrderDate
    ,OrderMonth = DATEFROMPARTS(YEAR(OrderDate),MONTH(OrderDate),1)
    ,TotalDue
    ,OrderRank = ROW_NUMBER() OVER(PARTITION BY DATEFROMPARTS(YEAR(OrderDate),MONTH(OrderDate),1) ORDER BY TotalDue DESC)
    FROM AdventureWorks2019.Purchasing.PurchaseOrderHeader
-- insert new data on temp table 2 for #Orders
INSERT INTO #OrdersMinusTop10 (
     OrderMonth
    ,OrderType
    ,TotalDue
    )
SELECT
    OrderMonth,
    OrderType = 'Purchase',
    TotalDue = SUM(TotalDue)
    FROM #Orders
    WHERE OrderRank > 10
    GROUP BY OrderMonth

-- use of temp tables
SELECT
    A.OrderMonth,
    TotalSales = A.TotalDue,
    TotalPurchases = B.TotalDue

    FROM #OrdersMinusTop10 A
        JOIN #OrdersMinusTop10 B
            ON A.OrderMonth = B.OrderMonth
                AND B.OrderType = 'Purchase'
    WHERE A.OrderType = 'Sales'
    ORDER BY 1

--  DROP tem tables
DROP TABLE #Orders
DROP TABLE #OrdersMinusTop10
```

![Screenshot](/assets/tp_truncate.png?raw=true "temporary tables truncate")

</details>

<details>
<summary><h3 style="color: #0063B2FF;">4. UPDATE()</h3></summary>

```SQL
CREATE TABLE #SalesOrders (
    SalesOrderID INT,
    OrderDate DATE,
    TaxAmt MONEY,
    Freight MONEY,
    TotalDue MONEY,
    TaxFreightPercent FLOAT,
    TaxFreightBucket VARCHAR(50),
    OrderAmtBucket VARCHAR(50),
    OrderCategory VARCHAR(50),
    OrderSubcategory VARCHAR(50)
    )
INSERT INTO #SalesOrders (
    SalesOrderID,
    OrderDate,
    TaxAmt,
    Freight,
    TotalDue,
    OrderCategory
    )
SELECT -- insert data
    SalesOrderID,
    OrderDate,
    TaxAmt,
    Freight,
    TotalDue,
    OrderCategory = 'Non-holiday Order'

    FROM [AdventureWorks2019].[Sales].[SalesOrderHeader]
    WHERE YEAR(OrderDate) = 2013

UPDATE #SalesOrders -- TaxFreightPercent
    SET
    TaxFreightPercent = (TaxAmt + Freight)/TotalDue,
    OrderAmtBucket =
        CASE
            WHEN TotalDue < 100 THEN 'Small'
            WHEN TotalDue < 1000 THEN 'Medium'
            ELSE 'Large'
        END

UPDATE #SalesOrders -- TaxFreightBucket
    SET TaxFreightBucket =
        CASE
            WHEN TaxFreightPercent < 0.1 THEN 'Small'
            WHEN TaxFreightPercent < 0.2 THEN 'Medium'
            ELSE 'Large'
        END

UPDATE #SalesOrders -- OrderCategory
    SET  OrderCategory = 'Holiday'
    FROM #SalesOrders
    WHERE DATEPART(quarter,OrderDate) = 4

UPDATE #SalesOrders -- OrderSubcategory
    SET  OrderSubcategory = OrderCategory + ' - ' + OrderAmtBucket

SELECT * FROM #SalesOrders
DROP TABLE #SalesOrders
```

![Screenshot](/assets/tp_update.png?raw=true "temporary tables update")

</details>

<details>
<summary><h3 style="color: #0063B2FF;">5. DELETE()</h3></summary>

```sql

```

![Screenshot](/assets/.png?raw=true "temporary tables delete")

</details>
