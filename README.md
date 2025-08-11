

INNER JOIN

Query

SELECT 
    c.CustomerID,
    c.Name AS CustomerName,
    o.OrderID,
    o.OrderDate,
    o.TotalAmount
FROM Customers c
INNER JOIN Orders o 
    ON c.CustomerID = o.CustomerID;

What it does (plain English)
Return only rows that have matching CustomerID in both Customers and Orders. If either side lacks a match, that row is not returned.

Expected result using your data

CustomerID	CustomerName	OrderID	     OrderDate	 TotalAmount

1	           John Doe	       1	     2025-08-01	   25500.00
2	          Alice Smith 	   2	      2025-08-02	  500.00


Why those rows?
Customers 1 and 2 each have orders. Customer 3 (Bob Brown) has no order, so he’s excluded.

Notes

Useful when you only need matched data (e.g., list customers who actually placed orders).

If the relation is one-to-many (e.g., Orders -> OrderDetails), an INNER JOIN will duplicate master-row values for each matching child row.


LEFT JOIN

Query

SELECT 
    c.CustomerID,
    c.Name AS CustomerName,
    o.OrderID,
    o.OrderDate,
    o.TotalAmount
FROM Customers c
LEFT JOIN Orders o 
    ON c.CustomerID = o.CustomerID;

What it does (plain English)
Return all rows from the left table (Customers) and matched rows from Orders. If there is no match, the Orders columns are NULL.

Expected result using your data

CustomerID	CustomerName	OrderID  	OrderDate	  TotalAmount

1          	John Doe    	1	       2025-08-01	    25500.00
2	          Alice Smith	  2        	2025-08-02  	500.00
3	          Bob Brown    	NULL	      NULL	        NULL


Why those rows?
All customers are returned. Bob Brown has no order so the order columns are NULL.

Common uses

Show master rows even when children don’t exist (e.g., show all customers and any orders they may have).

Combine with COALESCE() to replace nulls (COALESCE(o.TotalAmount, 0)).


RIGHT JOIN

Query

SELECT 
    o.OrderID,
    o.OrderDate,
    o.TotalAmount,
    c.CustomerID,
    c.Name AS CustomerName
FROM Customers c
RIGHT JOIN Orders o 
    ON c.CustomerID = o.CustomerID;

What it does (plain English)
Return all rows from the right table (Orders) and matched rows from Customers. If an order has no matching customer, Customers columns are NULL.

Expected result using your data

OrderID	OrderDate    	TotalAmount	CustomerID	CustomerName

1	      2025-08-01  	25500.00	  1	          John Doe
2      	2025-08-02	  500.00	    2	        Alice Smith


Why those rows?
All orders have matching customers in your dataset. If you had an Orders row with CustomerID missing or pointing to a non-existent customer, that order would still appear with customer columns NULL.

Note
MySQL supports RIGHT JOIN but many developers prefer LEFT JOIN and swap table order for readability (since LEFT JOIN is more common).

FULL JOIN (FULL OUTER JOIN) — MySQL notes & emulation

MySQL does not support FULL OUTER JOIN directly. A full join returns all rows from both tables, matching where possible and using NULL where there is no match.

Conceptual SQL (not supported in MySQL)

SELECT ... FROM Customers FULL JOIN Orders ON ...

Common emulation in MySQL (LEFT UNION RIGHT)

SELECT 
    c.CustomerID, c.Name, o.OrderID, o.OrderDate, o.TotalAmount
FROM Customers c
LEFT JOIN Orders o ON c.CustomerID = o.CustomerID

UNION

SELECT 
    c.CustomerID, c.Name, o.OrderID, o.OrderDate, o.TotalAmount
FROM Customers c
RIGHT JOIN Orders o ON c.CustomerID = o.CustomerID;

UNION (not UNION ALL) removes duplicate rows that appear in both halves (i.e., matched rows).

Alternatively, to avoid relying on UNION deduplication you can LEFT JOIN + UNION ALL selecting only the right-side-only rows:


SELECT c.CustomerID, c.Name, o.OrderID, o.OrderDate, o.TotalAmount
FROM Customers c
LEFT JOIN Orders o ON c.CustomerID = o.CustomerID

UNION ALL

SELECT c.CustomerID, c.Name, o.OrderID, o.OrderDate, o.TotalAmount
FROM Customers c
RIGHT JOIN Orders o ON c.CustomerID = o.CustomerID
WHERE c.CustomerID IS NULL;

This second pattern explicitly adds only those Orders rows that had no Customers match.

Result for your current data
Because every Order references an existing Customer, the LEFT JOIN already returned all customers (including the unmatched customer 3). The UNION adds nothing new — the full join result is the same as the LEFT JOIN result in this dataset.

Row duplication and one-to-many joins (important)

If you join Customers -> Orders -> OrderDetails -> Products you will see repeated customer/order values when there are multiple order details per order.

Example query

SELECT 
  o.OrderID, c.Name, od.OrderDetailID, p.ProductName, od.Quantity, od.Price
FROM Orders o
JOIN Customers c ON o.CustomerID = c.CustomerID
JOIN OrderDetails od ON od.OrderID = o.OrderID
JOIN Products p ON od.ProductID = p.ProductID;

Expected rows (your data):

Order 1 (John Doe) has two order details → 2 rows:

(1, John Doe, ..., Smartphone, 1, 25000.00)

(1, John Doe, ..., T-Shirt, 1, 500.00)


Order 2 (Alice Smith) has one detail → 1 row:

(2, Alice Smith, ..., T-Shirt, 1, 500.00)



If you want one row per order with the total computed from details, use aggregation:

SELECT o.OrderID, c.Name,
       SUM(od.Quantity * od.Price) AS ComputedTotal
FROM Orders o
JOIN Customers c ON o.CustomerID = c.CustomerID
JOIN OrderDetails od ON od.OrderID = o.OrderID
GROUP BY o.OrderID, c.Name;

This ComputedTotal should match Orders.TotalAmount if details are consistent.

NULLs and join keys

Joins match on exact values. NULL does not equal NULL in SQL. So an ON condition like c.CustomerID = o.CustomerID will not match rows where either side is NULL.

Example: if an Orders row had CustomerID = NULL, an INNER JOIN or LEFT JOIN ON equality will not match any customer (it will appear as an unmatched row on the Orders side).


Performance & indexing tips

Index the join columns for faster joins:

Orders(CustomerID) — helps joins from Customers -> Orders.

OrderDetails(OrderID), OrderDetails(ProductID).

Many engines automatically index foreign keys, but confirm indexes exist.


For large joins prefer:

Explicit SELECT column lists (don’t SELECT * unnecessarily).

Proper filtering in WHERE to reduce row counts before grouping or further joins.

EXPLAIN to see the query plan (EXPLAIN SELECT ...) and check for full table scans.



Common mistakes to avoid

Expecting FULL JOIN in MySQL — remember to emulate it.

Not accounting for duplicates in one-to-many relationships.

Using UNION ALL instead of UNION when you want to remove duplicates (or vice versa).

Relying on implicit column order when using UNION — ensure both SELECT parts return the same number and compatible column types.








Which follow-up would help you most?
