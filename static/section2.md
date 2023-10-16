## ⭐ <span style="color: #0063B2FF;">SQL SERVER - SECTION 2</span>⭐

## 🔗 <span style="color: #0063B2FF;">WINDOW FUNCTIONS</span>

Window functions operate on a set of rows and return a single value for each row from the underlying query. The term <b style="color: #ee6c4d;">window</b> describes the set of rows on which the function operates. A <b style="color: #ee6c4d;">window function</b> uses values from the rows in a window to calculate the returned values.

<details>
 <summary><h3 style="color: #0063B2FF;">1. OVER()</h3>

The `OVER()` clause (window definition) differentiates window functions from other analytical and reporting functions. A query can include multiple window functions with the same or different window definitions.

The `OVER()` clause has the following capabilities:

- Defines window partitions to form groups of rows (`PARTITION BY` clause).
- Orders rows within a partition (`ORDER BY` clause).

Aggregate function that can be used on window functions:

- `COUNT()`
- `AVG()`
- `SUM()`
- `MAX()`
- `MIN()`
- `FIRST_VALUE()`
- `LAST_VALUE()`
- `CUME_DIST()`
- `LAG()`
- `LEAD()`
- `ROW_NUMBER()`
- `RANK()`
- `DENSE_RANK()`
- `PERCENT_RANK()`
- `NTILE()`

</summary>

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

</details>

<details>
 <summary><h3 style="color: #0063B2FF;">2. PARTITION()</h3>

`PARTITION BY` is an optional clause inside the `OVER()` clause that <span style="color: #ee6c4d;">subdivides the data into partitions</span>. Including the partition clause divides the query result set into partitions, and the window function is applied to each partition separately. Computation restarts for each partition. If you do not include a partition clause, the function calculates on the entire table or file.

</summary>

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

</details>

<details>
 <summary><h3 style="color: #0063B2FF;">3. RANKING WINDOW: ROW_NUMBER()</h3>

The `ROW_NUMBER()` window function determines the ordinal number of the current row within its partition. The `ORDER BY` expression in the `OVER()` clause determines the number. Each value is ordered within its partition. Rows with equal values for the `ORDER BY` expressions receive different row numbers non-deterministically.

</summary>

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

</details>

<details>
 <summary><h3 style="color: #0063B2FF;">4. RANKING WINDOW: RANK(), DENSE_RANK()</h3>

The `RANK()` window function determines the rank of a value in a group of values. The `ORDER BY` expression in the `OVER()` clause determines the value. Each value is ranked within its partition. Rows with equal values for the ranking criteria receive the same rank. Drill adds the number of tied rows to the tied rank to calculate the next rank and thus the ranks might not be consecutive numbers.
<br>
For example, if two rows are ranked 1, the next rank is 3. The `DENSE_RANK()` window function differs in that no gaps exist if two or more rows tie.

The `DENSE_RANK()` window function determines the rank of a value in a group of values based on the `ORDER BY` expression and the `OVER()` clause. Each value is ranked within its partition. Rows with equal values receive the same rank. There are no gaps in the sequence of ranked values if two or more rows have the same rank.

</summary>

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

</details>

<details>
 <summary><h3 style="color: #0063B2FF;">5. VALUE WINDOW: LEAD(), LAG(), FIRST_VALUE(), LAST_VALUE()</h3>

The `LAG()` window function returns the value for the <b style="color: #ee6c4d;">row before the current row</b> in a partition. If no row exists, null is returned.

The `LEAD()` window function returns the value for the <b style="color: #ee6c4d;">row after the current row</b> in a partition. If no row exists, null is returned.

</summary>

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

</details>

<details>
 <summary><h3 style="color: #0063B2FF;">6. SUB-QUERIES</h3>

A <b style="color: #ee6c4d;">subquery</b> is a query that is nested inside a `SELECT`, `INSERT`, `UPDATE`, or `DELETE` statement, or inside another subquery.
Usually when the first query has window functions and requires some conditions such as `WHERE` clauses.

</summary>

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

</details>
