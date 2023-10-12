## ⭐ SQL SERVER - SECTION 4 ⭐

## 🔗 CTE's Common Table Expressions:

The **common table expression (CTE)** is a powerful construct in SQL that helps simplify a query. CTEs work as **virtual tables** (with records and columns), created during the execution of a query, used by the query, and eliminated after query execution. CTEs often act as a bridge to transform the data in source tables to the format expected by the query.
<br>
A common table expression, or CTE, is a temporary named result set created from a simple SELECT statement that can be used in a subsequent SELECT statement. Each SQL CTE is like a named query, whose result is stored in a virtual table (a CTE) to be referenced later in the main query.
<br>
The best way to learn common table expressions is through practice. I recommend LearnSQL.com's interactive Recursive Queries course. It contains over 100 exercises that teach CTEs starting with the basics and progressing to advanced topics like recursive common table expressions.
<br>
Topics of this section:

- 1. CTE
- 2. Recursive CTE

### 1. CTE COMMON TABLE EXPRESSION

You can define two or more CTEs and use them in the main query. In the next example, we show you how to divide and organize a long query using SQL CTEs. By naming different parts of the query, CTEs make the query easy to read.

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

### 2. RECURSIVE CTE

SQL Server offers **recursive Common Table Expressions**. A recursive Common Table Expression (CTE) in SQL Server allows you to perform recursive queries on hierarchical or graph-based data structures, such as organizational charts, family trees, transportation networks, etc. Recursive queries are used to **loop** through relationships between the data elements.

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
