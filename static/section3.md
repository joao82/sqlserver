## ⭐ <span style="color: #0063B2FF;">SQL SERVER - SECTION 3</span>⭐

## 🔗 <span style="color: #0063B2FF;">SUB-QUERIES</span>

A sub-query is also called an <b style="color: #ee6c4d;">inner query</b> or <b style="color: #ee6c4d;">inner select</b>, while the statement containing a sub-query is also called an <b style="color: #ee6c4d;">outer query</b> or <b style="color: #ee6c4d;">outer select</b>.
Many Transact-SQL statements that include sub-queries can be alternatively formulated as joins. Other questions can be posed only with sub-queries. In Transact-SQL, there's usually no performance difference between a statement that includes a sub-query and a semantically equivalent version that doesn't. For architectural information on how SQL Server processes queries, see SQL statement processing. However, in some cases where existence must be checked, a join yields better performance. Otherwise, the nested query must be processed for each result of the outer query to ensure elimination of duplicates. In such cases, a join approach would yield better results.

<details>
 <summary><h3 style="color: #0063B2FF;">1. SCALAR SUB-QUERIES</h3>

A <b style="color: #ee6c4d;">scalar sub-query</b> is a sub-query that selects only one column or expression and returns one row. A scalar sub-query can be used anywhere in an SQL query that a column or expression can be used.

</summary>

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

![Screenshot](/assets/over.png?raw=true "over")

</details>

<details>
 <summary><h3 style="color: #0063B2FF;">2. CORRELATED SUB-QUERIES</h3></summary>

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

</details>

<details>
 <summary><h3 style="color: #0063B2FF;">3. EXISTS()</h3>

The <b style="color: #ee6c4d;">EXISTS</b> operator is used to test for the existence of any record in a subquery.
<br>
The <b style="color: #ee6c4d;">EXISTS</b> operator returns TRUE if the subquery returns one or more records.

</summary>

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

</details>

<details>
 <summary><h3 style="color: #0063B2FF;">4. FOR XML PATH()</h3>

In SQL Server, <b style="color: #ee6c4d;">STUFF</b> with <b style="color: #ee6c4d;">SELECT FOR XML PATH</b> statement is widely used to concatenate strings from multiple rows into a single row value.

</summary>

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

</details>

<details>
 <summary><h3 style="color: #0063B2FF;">5. PIVOT()</h3>

You can use the <b style="color: #ee6c4d;">PIVOT</b> and <b style="color: #ee6c4d;">UNPIVOT</b> relational operators to change a table-valued expression into another table. <b style="color: #ee6c4d;">PIVOT</b> rotates a table-valued expression by turning the unique values from one column in the expression into multiple columns in the output. And PIVOT runs aggregations where they're required on any remaining column values that are wanted in the final output. <b style="color: #ee6c4d;">UNPIVOT</b> carries out the opposite operation to PIVOT by rotating columns of a table-valued expression into column values.

</summary>

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

</details>
