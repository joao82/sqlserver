## ⭐ SQL SERVER ⭐

This repository contains the notes of the advanced SQL tutorial: **The Advanced SQL Server Masterclass For Data Analysis** from **FreeCodeCamp** and **Developedbyed** on Youtube, as well as individual capstone projects by using the following languages **HTML5, Sass, CSS3, Tailwindcss, JavaScript, NodeJs, Django, React and Redux**.

Here's a look at just some of the things you'll get out of this course:

- Make the leap to Senior Analyst by mastering advanced data wrangling techniques with SQL
- Become the resident SQL expert on your team
- Perform nuanced analysis of large datasets with Window Functions
- Use subqueries, CTEs and temporary tables to handle complex, multi-stage queries and data transformations
- Write efficient, optimized SQL
- Leverage indexes to speed up your SQL queries
- Supercharge your SQL knowledge with procedural programming techniques like variables and IF statements
- Program database objects like user defined functions and stored procedures that will make life easier for you AND your teammates
- Master useful tips and tricks not found in most database courses, like Dynamic SQL
- Gain an intuition for what technique to apply and when
- Train your brain with tons of hands-on exercises that reflect real-world business scenarios

## 🔗 Window functions:

Window functions operate on a set of rows and return a single value for each row from the underlying query. The term **window** describes the set of rows on which the function operates. A window function uses values from the rows in a window to calculate the returned values.

### 1. OVER()

The <b>OVER()</b> clause (window definition) differentiates window functions from other analytical and reporting functions. A query can include multiple window functions with the same or different window definitions.

The OVER() clause has the following capabilities: Defines **window partitions** to form groups of rows (PARTITION BY clause).
Orders rows within a partition (ORDER BY clause).

Aggregate function that can be used on window functions:
AVG()
COUNT()
CUME_DIST()
DENSE_RANK()
FIRST_VALUE()
LAG()
LAST_VALUE()
LEAD()
MAX()
MIN()
NTILE()
PERCENT_RANK()
RANK()
ROW_NUMBER()
SUM()

```sql
SELECT
     B.FirstName [First Name]
    ,B.LastName [Last Name]
    ,A.Rate Rate
    ,AverageRate = AVG(A.Rate) OVER()
    ,MaximumRate = MAX(A.Rate) OVER()
    ,DiffFromAvgRate = A.Rate - AVG(A.Rate) OVER()
    ,PercentageofMaxRAte = A.Rate/MAX(A.Rate) OVER()

    FROM AdventureWorks2019.Person.Person B
        JOIN AdventureWorks2019.HumanResources.EmployeePayHistory A
            ON A.BusinessEntityID = B.BusinessEntityID
```

![Screenshot](/assets/over.png?raw=true "over")

### 2. PARTITION()

**PARTITION BY** expr_list PARTITION BY is an optional clause that subdivides the data into partitions. Including the partition clause divides the query result set into partitions, and the window function is applied to each partition separately. Computation restarts for each partition. If you do not include a partition clause, the function calculates on the entire table or file.

```sql
SELECT
     ProductName = A.Name
    ,A.ListPrice
    ,ProductCategory = C.Name
    ,ProductSubcategory = B.Name
    ,AvgPriceByCategory = AVG(A.ListPrice) OVER(PARTITION BY C.Name)
    ,AvgPriceByCategoryAndSubcategory = AVG(A.ListPrice) OVER(PARTITION BY C.Name, B.Name)
    ,ProductVsCategoryDelta = A.ListPrice - AVG(A.ListPrice) OVER(PARTITION BY C.Name)

    FROM AdventureWorks2019.Production.Product A
        JOIN AdventureWorks2019.Production.ProductSubcategory B
            ON A.ProductSubcategoryID = B.ProductSubCategoryID
        JOIN AdventureWorks2019.Production.ProductCategory C
            ON B.ProductCategoryID = C.ProductCategoryID
```

![Screenshot](/assets/partition.png?raw=true "partition")

### 3. RANKING: ROW_NUMBER()

The ROW_NUMBER window function determines the ordinal number of the current row within its partition. The ORDER BY expression in the OVER clause determines the number. Each value is ordered within its partition. Rows with equal values for the ORDER BY expressions receive different row numbers non-deterministically.

```sql
SELECT
     ProductName = A.Name
    ,Price = A.ListPrice
    ,ProductSubcategory = B.Name
    ,ProductCategory = C.Name
    ,[Price Rank] = ROW_NUMBER() OVER(ORDER BY A.ListPrice DESC)
    ,[Category Price Rank] = ROW_NUMBER() OVER(PARTITION BY C.Name ORDER BY A.ListPrice DESC)
    ,[TOP 5 Price in Category] =
        CASE
            WHEN ROW_NUMBER() OVER(PARTITION BY C.Name ORDER BY A.ListPrice DESC) <= 5 THEN 'Yes'
            ELSE 'No'
        END

    FROM AdventureWorks2019.Production.Product A
        JOIN AdventureWorks2019.Production.ProductSubcategory B
            ON A.ProductSubcategoryID = B.ProductSubCategoryID
        JOIN AdventureWorks2019.Production.ProductCategory C
            ON B.ProductCategoryID = C.ProductCategoryID

    ORDER BY  [TOP 5 Price in Category] DESC
```

![Screenshot](/assets/row_number.png?raw=true "row number")

### 4. RANKING WINDOW: RANK() DENSE_RANK() PERCENT_RANK() NTILE() CUME_DIST()

The **RANK()** window function determines the rank of a value in a group of values. The ORDER BY expression in the OVER clause determines the value. Each value is ranked within its partition. Rows with equal values for the ranking criteria receive the same rank. Drill adds the number of tied rows to the tied rank to calculate the next rank and thus the ranks might not be consecutive numbers. For example, if two rows are ranked 1, the next rank is 3. The DENSE_RANK window function differs in that no gaps exist if two or more rows tie.

The **DENSE_RANK()** window function determines the rank of a value in a group of values based on the ORDER BY expression and the OVER clause. Each value is ranked within its partition. Rows with equal values receive the same rank. There are no gaps in the sequence of ranked values if two or more rows have the same rank.

```SQL
SELECT
     ProductName = A.Name
    ,A.ListPrice
    ,ProductSubcategory = B.Name
    ,ProductCategory = C.Name
    ,[Price Rank] = ROW_NUMBER() OVER(ORDER BY A.ListPrice DESC)
    ,[Category Price Rank] = ROW_NUMBER() OVER(PARTITION BY C.Name ORDER BY A.ListPrice DESC)
    ,[Category Price Rank with Rank] = RANK() OVER(PARTITION BY C.Name ORDER BY A.ListPrice DESC)
    ,[Category Price Rank with Dense Rank] = DENSE_RANK() OVER(PARTITION BY C.Name ORDER BY A.ListPrice DESC)
    ,[TOP 5 Price in Category] =
        CASE
            WHEN DENSE_RANK() OVER(PARTITION BY C.Name ORDER BY A.ListPrice DESC) <= 5 THEN 'Yes'
        ELSE 'No'
        END

    FROM AdventureWorks2019.Production.Product A
        JOIN AdventureWorks2019.Production.ProductSubcategory B
            ON A.ProductSubcategoryID = B.ProductSubCategoryID
        JOIN AdventureWorks2019.Production.ProductCategory C
            ON B.ProductCategoryID = C.ProductCategoryID

    ORDER BY  [TOP 5 Price in Category] DESC
```

![Screenshot](/assets/rank.png?raw=true "rank")

### 5. VALUE WINDOW: LEAD() LAG() FIRST_VALUE() LAST_VALUE()

The **LAG()** window function returns the value for the row before the current row in a partition. If no row exists, null is returned.

The **LEAD()** window function returns the value for the row after the current row in a partition. If no row exists, null is returned.

```sql
SELECT
     A.PurchaseOrderID
    ,A.OrderDate
    ,A.TotalDue
    ,VendorName = B.Name
    ,PrevOrderAmtFromVendor = LAG(A.TotalDue) OVER(PARTITION BY A.VendorID ORDER BY A.OrderDate)
    ,NextOrderByEmployeeVendor = LEAD(B.Name) OVER(PARTITION BY A.EmployeeID ORDER BY A.OrderDate)
    ,Next2OrderByEmployeeVEndor = LEAD(B.Name, 2) OVER(PARTITION BY A.EmployeeID ORDER BY A.OrderDate)
    ,A.VendorID
    ,A.EmployeeID

    FROM AdventureWorks2019.Purchasing.PurchaseOrderHeader A
        JOIN AdventureWorks2019.Purchasing.Vendor B
            ON A.VendorID = B.BusinessEntityID

    WHERE YEAR(A.OrderDate) >= 2013
    AND A.TotalDue > 500

      ORDER BY A.EmployeeID, A.OrderDate
```

![Screenshot](/assets/lead.png?raw=true "lead")

### 6. SUB-QUERIES

A **subquery** is a query that is nested inside a SELECT, INSERT, UPDATE, or DELETE statement, or inside another subquery.
Usually when the first query has window functions and requires some conditions such as WHERE clauses.

```sql
SELECT
     PurchaseOrderID
    ,VendorID
    ,OrderDate
    ,TaxAmt
    ,Freight
    ,TotalDue

    FROM
        (SELECT
             PurchaseOrderID
            ,VendorID
            ,OrderDate
            ,TaxAmt
            ,Freight
            ,TotalDue
            ,[Top 3 Purchase Order Rank] = ROW_NUMBER() OVER(PARTITION BY VendorID ORDER BY TotalDue DESC)
            ,[All Top 3 Purchase Order Rank] = DENSE_RANK() OVER(PARTITION BY VendorID ORDER BY TotalDue DESC)

            FROM AdventureWorks2019.Purchasing.PurchaseOrderHeader
        ) X
    -- Windowed functions can only appear in the SELECT or ORDER BY clauses.
    WHERE AllTop3PurchaseOrderRank <= 3
```
