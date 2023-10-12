## ⭐ SQL SERVER - SECTION 3 ⭐

## 🔗 SUB-QUERIES:

A sub-query is also called an **inner query** or **inner select**, while the statement containing a sub-query is also called an **outer query** or **outer select**.
Many Transact-SQL statements that include sub-queries can be alternatively formulated as joins. Other questions can be posed only with sub-queries. In Transact-SQL, there's usually no performance difference between a statement that includes a sub-query and a semantically equivalent version that doesn't. For architectural information on how SQL Server processes queries, see SQL statement processing. However, in some cases where existence must be checked, a join yields better performance. Otherwise, the nested query must be processed for each result of the outer query to ensure elimination of duplicates. In such cases, a join approach would yield better results.

Topics of this section:

- 1. Scalar Sub-queries
- 2. Correlated Sub-queries
- 3. Exists
- 4. For XML path
- 5. Pivot

### 1. SCALAR SUB-QUERIES

A **scalar sub-query** is a sub-query that selects only one column or expression and returns one row. A scalar sub-query can be used anywhere in an SQL query that a column or expression can be used.

```sql
SELECT
     BusinessEntityID
    ,JobTitle
    ,VacationHours
    ,MaxVacationHours = (SELECT MAX(VacationHours) FROM AdventureWorks2019.HumanResources.Employee)
    ,PercentOfMaxVacationHours = VacationHours / (SELECT MAX(VacationHours) FROM AdventureWorks2019.HumanResources.Employee)

    FROM AdventureWorks2019.HumanResources.Employee

    WHERE VacationHours / (SELECT MAX(VacationHours) FROM AdventureWorks2019.HumanResources.Employee) >= 0.8
```

<!-- ![Screenshot](/assets/over.png?raw=true "over") -->

### 2. CORRELATED SUB-QUERIES

```sql
SELECT
     PurchaseOrderID
    ,VendorID
    ,OrderDate
    ,TotalDue
    ,NonRejectedItems = -- correlated subquery
    (
        SELECT
            COUNT(*)
            FROM AdventureWorks2019.Purchasing.PurchaseOrderDetail B
            WHERE A.PurchaseOrderID = B.PurchaseOrderID
            AND B.RejectedQty = 0
    )
    ,MostExpensiveItem = -- correlated subquery
    (
        SELECT
            MAX(B.UnitPrice)
            FROM AdventureWorks2019.Purchasing.PurchaseOrderDetail B
            WHERE A.PurchaseOrderID = B.PurchaseOrderID
    )

    FROM AdventureWorks2019.Purchasing.PurchaseOrderHeader A
```

![Screenshot](/assets/correlated_sub.png?raw=true "correlated sub-queries")

### 3. EXISTS()

The **EXISTS** operator is used to test for the existence of any record in a subquery.
The **EXISTS** operator returns TRUE if the subquery returns one or more records.

```sql
SELECT
    A.PurchaseOrderID,
    A.OrderDate,
    A.SubTotal,
    A.TaxAmt

    FROM AdventureWorks2019.Purchasing.PurchaseOrderHeader A
    WHERE EXISTS (
        SELECT
            1
            FROM AdventureWorks2019.Purchasing.PurchaseOrderDetail B
            WHERE A.PurchaseOrderID = B.PurchaseOrderID
            AND B.OrderQty > 500
            AND B.UnitPrice > 50
    )
    ORDER BY A.PurchaseOrderID ASC
```

![Screenshot](/assets/exists.png?raw=true "exists")

### 4. FOR XML PATH()

In SQL Server, **STUFF** with **SELECT FOR XML PATH** statement is widely used to concatenate strings from multiple rows into a single row value.

```SQL
SELECT
    SubcategoryName = A.[Name]
    ,Products =
    STUFF(  -- Concatenate values using ';' delimiter; STUFF function is only used to remove the leading (,)

        (
            SELECT
                ';' + B.Name

            FROM AdventureWorks2019.Production.Product B

            WHERE A.ProductSubcategoryID = B.ProductSubcategoryID
                AND B.ListPrice > 50

            FOR XML PATH('')

        ), 1, 1, ''
    )
    FROM AdventureWorks2019.Production.ProductSubcategory A
```

![Screenshot](/assets/xml_path.png?raw=true "xml_path")

### 5. PIVOT()

You can use the **PIVOT** and **UNPIVOT** relational operators to change a table-valued expression into another table. **PIVOT** rotates a table-valued expression by turning the unique values from one column in the expression into multiple columns in the output. And PIVOT runs aggregations where they're required on any remaining column values that are wanted in the final output. UNPIVOT carries out the opposite operation to PIVOT by rotating columns of a table-valued expression into column values.

```sql
SELECT
    [Order Quantity] = OrderQty,
    [Accessories],
    [Bikes],
    [Clothing],
    [Components]

    FROM
        (
        SELECT
            ProductCategoryName = D.Name,
            A.LineTotal,
            A.OrderQty

        FROM AdventureWorks2019.Sales.SalesOrderDetail A
            JOIN AdventureWorks2019.Production.Product B
                ON A.ProductID = B.ProductID
            JOIN AdventureWorks2019.Production.ProductSubcategory C
                ON B.ProductSubcategoryID = C.ProductSubcategoryID
            JOIN AdventureWorks2019.Production.ProductCategory D
                ON C.ProductCategoryID = D.ProductCategoryID
        ) E

    PIVOT(
        SUM(LineTotal)
        FOR ProductCategoryName IN([Accessories],[Bikes],[Clothing],[Components])
    ) F

    ORDER BY 1
```

![Screenshot](/assets/pivot.png?raw=true "pivot")
