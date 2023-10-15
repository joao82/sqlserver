## ⭐ <span style="color: #0063B2FF;">SQL SERVER - SECTION 4</span>⭐

## 🔗 <span style="color: #0063B2FF;">CTE - COMMON TABLE EXPRESSIONS</span>

<span style="color: #0063B2FF;"></span>
<span style="color: #ee6c4d;"></span>

The <span style="color: #ee6c4d;">**Common Table Expression - CTE**</span> is a powerful construct in SQL that helps simplify a query. CTEs work as <span style="color: #ee6c4d;">**virtual tables**</span> (with records and columns), created during the execution of a query, used by the query, and eliminated after query execution. CTEs often act as a <span style="color: #ee6c4d;">bridge</span> to transform the data in source tables to the format expected by the query.
<br>
<br>
A <span style="color: #ee6c4d;">**Common Table Expression - CTE**</span> is a temporary named result set created from a simple SELECT statement that can be used in a subsequent SELECT statement. Each SQL CTE is like a named query, whose result is stored in a virtual table (a CTE) to be referenced later in the main query.

<details>
 <summary><h3 style="color: #0063B2FF;">1. CTE COMMON TABLE EXPRESSION</h3>
 
You can define two or more CTEs and use them in the main query. In the next example, we show you how to divide and organize a long query using SQL CTEs. By naming different parts of the query, CTEs make the query easy to read.
 </summary>

```sql
WITH
    -- 1st CTE
    sales AS (
        SELECT
            OrderDate,
            OrderMonth = DATEFROMPARTS(YEAR(OrderDate),MONTH(OrderDate),1),
            TotalDue,
            OrderRank = ROW_NUMBER() OVER(PARTITION BY DATEFROMPARTS(YEAR(OrderDate),MONTH(OrderDate),1) ORDER BY TotalDue DESC)

            FROM AdventureWorks2019.Sales.SalesOrderHeader
    ),
    -- 2nd CTE
    SalesMinusTop10 AS (
        SELECT
            OrderMonth,
            TotalSales = SUM(TotalDue)

            FROM Sales -- using 1st CTE
            WHERE OrderRank > 10
            GROUP BY OrderMonth
    )
    -- 3rd CTE
    ,Purchases AS (
        SELECT
            OrderDate
            ,OrderMonth = DATEFROMPARTS(YEAR(OrderDate),MONTH(OrderDate),1)
            ,TotalDue
            ,OrderRank = ROW_NUMBER() OVER(PARTITION BY DATEFROMPARTS(YEAR(OrderDate),MONTH(OrderDate),1) ORDER BY TotalDue DESC)

            FROM AdventureWorks2019.Purchasing.PurchaseOrderHeader
    )
    -- 4rd CTE
    ,PurchasesMinusTop10 AS (
        SELECT
            OrderMonth,
            TotalPurchases = SUM(TotalDue)

            FROM Purchases -- using 3rd CTE
            WHERE OrderRank > 10
            GROUP BY OrderMonth
    )

    SELECT
        A.OrderMonth,
        A.TotalSales,
        B.TotalPurchases

        FROM SalesMinusTop10 A -- using 2nd CTE
            LEFT JOIN PurchasesMinusTop10 B -- using 4rd CTE
                ON A.OrderMonth = B.OrderMonth

        ORDER BY 1
```

![Screenshot](/assets/cte.png?raw=true "cte")

</details>

<details>
<summary><h3 style="color: #0063B2FF;">2. RECURSIVE CTE</h3>

SQL Server offers <span style="color: #ee6c4d;">**Recursive CTE Common Table Expressions**</span>. A recursive Common Table Expression (CTE) in SQL Server allows you to perform recursive queries on hierarchical or graph-based data structures, such as organizational charts, family trees, transportation networks, etc.
<br>
<br>
Recursive queries are used to <span style="color: #ee6c4d;">**loop**</span> through relationships between the data elements.

</summary>

```sql example 1
WITH
    -- CTE
    DayMonthDates AS (
        -- Anchor member
        SELECT CAST('01-01-2021' AS DATE) AS FirstMonth
        -- loop over
        UNION  ALL
        -- looping conditions
        SELECT
            DATEADD(MONTH, 1, FirstMonth)
            FROM DayMonthDates
            WHERE FirstMonth < CAST('12-31-2021' AS DATE)
    )
    -- CTE are limited to the 1st SELECT query after the CTE
    SELECT
        FirstMonth

        FROM DayMonthDates
        OPTION (MAXRECURSION 120)
```

```sql example 2
WITH
    DateSeries AS (
        -- Anchor member
        SELECT CAST('01-01-2021' AS DATE) AS MyDate
        -- loop over
        UNION  ALL
        -- looping conditions
        SELECT
            DATEADD(DAY, 1, MyDate)

            FROM DateSeries
            WHERE MyDate < CAST('12-31-2021' AS DATE)
    )

    SELECT
        MyDate
        FROM DateSeries
        OPTION (MAXRECURSION 365)
```

![Screenshot](/assets/recursive_cte.png?raw=true "recursive cte")

</details>
