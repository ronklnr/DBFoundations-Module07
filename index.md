# Assignment 7 - Writeup for Module 7

**Name:** Ron Klangnarong  
**Date:** March 1, 2022  
**GitHub URL:** https://github.com/ronklnr/DBFoundations-Module07

## Introduction
This write-up is part of the assignment for Module 7. It answers two questions: when to use an SQL UDF and the differences between Scalar, Inline, and Multi-Statement Functions.

## When to use an SQL UDF
UDFs, or user-defined functions, allow users to create custom functions. These functions perform a set of defined actions, such as calculations or formatting, then return values. UDFs are not only useful for recurring tasks, but users can use them to transform data, check constraints, and select reporting data.
 
## Differences between Scalar, Inline, and Multi-Statement Functions
UDFs have three types: Scalar Valued Functions, Inline Functions, and Multi-Statement Functions.
Scalar Valued Functions return a single value. Inline Functions and Multi-Statement Functions are table-valued functions. As the name suggests, they return a table of values. With Multi-Statement Functions, however, users can use multiple SELECT Statements to add actions. 

There are a few differences in syntax. Users must specify the type of functions by writing “RETURNS (data type)” for Scalar Functions and “RETURNS TABLE” for Inline Functions. For Multi-Statement Functions, users have to declare a table variable (e.g. RETURNS @t TABLE). Unlike Inline Functions and Multi-Statement Functions, Scalar Values Functions require a BEGIN/END block between the SELECT Statement. 

## Summary
With UDFs, or user-defined functions, users can repeatedly perform tasks without writing new code. UDFs can also be used for constraint checking and reporting purposes. Scalar Functions, one of the three UDF types, return a single values, while Inline and Multi-Statement Functions return a table of data. 

## Referece
Gould, A. (2013, Feb 20), SQL Server Programming Part 10 - Table Valued Function, WiseOwlTutorials, www.youtube.com/watch?v=nCAEgNxC7nU

# Assignment 7 - Writeup for Module 7
```
# ------------------------------------------------- #
# Title: Assignment7
## Author: Ron Klangnarong
## Desc: This file demonstrates how to use Functions
## Change Log: When,Who,What
# 2022-2-28, Ron Klangnarong, Created File
# ------------------------------------------------- #

-- Question 1 (5% of pts):
-- Show a list of Product names and the price of each product.
-- Use a function to format the price as US dollars.
-- Order the result by the product name.
-- <Put Your Code Here> --

SELECT 
      [Product Name] = ProductName
	 ,[Unit Price] = FORMAT(UnitPrice, 'C', 'en-US')
  FROM vProducts
  ORDER BY ProductName;
GO 

-- Question 2 (10% of pts): 
-- Show a list of Category and Product names, and the price of each product.
-- Use a function to format the price as US dollars.
-- Order the result by the Category and Product.

SELECT 
     [Category Name] = CategoryName
	,[Product Name] = ProductName
	,[Unit Price] = FORMAT(UnitPrice, 'C', 'en-US')
  FROM vCategories AS C 
   JOIN vProducts AS P 
   ON C.CategoryID = P.CategoryID
  ORDER BY CategoryName, ProductName;
GO 

-- Question 3 (10% of pts): 
-- Use functions to show a list of Product names, each Inventory Date, and the Inventory Count.
-- Format the date like 'January, 2017'.
-- Order the results by the Product and Date.

SELECT
     [Product Name] = ProductName
	,[Inventory Date] = DATENAME(MONTH, InventoryDate) + ', ' + DATENAME(YEAR, InventoryDate)
	,[Inventory Count] = Count 
  FROM vProducts AS P 
   JOIN vInventories AS I   
    ON P.ProductID = I.ProductID
  ORDER BY ProductName, MONTH(InventoryDate), YEAR(InventoryDate);
GO 

-- Question 4 (10% of pts): 
-- CREATE A VIEW called vProductInventories. 
-- Shows a list of Product names, each Inventory Date, and the Inventory Count. 
-- Format the date like 'January, 2017'.
-- Order the results by the Product and Date.

CREATE VIEW vProductInventories AS
 SELECT
 TOP 100000
	 [Product Name] = ProductName
	,[Inventory Date] = DATENAME(MONTH, InventoryDate) + ', ' + DATENAME(YEAR, InventoryDate)
	,[Inventory Count] = Count 
  FROM vProducts AS P 
   JOIN vInventories AS I   
    ON P.ProductID = I.ProductID
  ORDER BY ProductName, MONTH(InventoryDate), YEAR(InventoryDate);
GO 

-- Check that it works: Select * From vProductInventories;
SELECT * FROM vProductInventories;
GO 

-- Question 5 (10% of pts): 
-- CREATE A VIEW called vCategoryInventories. 
-- Shows a list of Category names, Inventory Dates, and a TOTAL Inventory Count BY CATEGORY
-- Format the date like 'January, 2017'.
-- Order the results by the Product and Date.

CREATE VIEW vCategoryInventories AS 
  SELECT 
    TOP 100000
	 [Category Name] = CategoryName
	,[Inventory Date] = DATENAME(MONTH, InventoryDate) + ', ' + DATENAME(YEAR, InventoryDate)
	,[Inventory Count] = SUM(Count) OVER(PARTITION BY CategoryName)
  FROM vCategories AS C  
   JOIN vProducts AS P
    ON C.CategoryID = P.CategoryID
   JOIN vInventories AS I  
    ON P.ProductID = I.ProductID
 ORDER BY CategoryName, MONTH(InventoryDate), YEAR(InventoryDate);
GO  

-- Check that it works: Select * From vCategoryInventories;
SELECT * FROM vCategoryInventories;
GO

-- Question 6 (10% of pts): 
-- CREATE ANOTHER VIEW called vProductInventoriesWithPreviouMonthCounts. 
-- Show a list of Product names, Inventory Dates, Inventory Count, AND the Previous Month Count.
-- Use functions to set any January NULL counts to zero. 
-- Order the results by the Product and Date. 
-- This new view must use your vProductInventories view.

CREATE VIEW vProductInventoriesWithPreviouMonthCounts AS 
  SELECT 
    TOP 100000
	 [Product Name]
	,[Inventory Date] 
	,[Inventory Count] = SUM([Inventory Count])
	,[Previous Month Count] = IIF(MONTH([Inventory Date]) = 1, 0,  
	   LAG(SUM([Inventory Count])) OVER (ORDER BY [Product Name], MONTH([Inventory Date]), YEAR([Inventory Date])))
  FROM  vProductInventories
  GROUP BY [Product Name], [Inventory Date] 
  ORDER BY [Product Name], MONTH([Inventory Date]), YEAR([INVENTORY Date]);
GO 

-- Check that it works: Select * From vProductInventoriesWithPreviousMonthCounts;
SELECT * FROM vProductInventoriesWithPreviouMonthCounts;
GO 

-- Question 7 (15% of pts): 
-- CREATE a VIEW called vProductInventoriesWithPreviousMonthCountsWithKPIs.
-- Show columns for the Product names, Inventory Dates, Inventory Count, Previous Month Count. 
-- The Previous Month Count is a KPI. The result can show only KPIs with a value of either 1, 0, or -1. 
-- Display months with increased counts as 1, same counts as 0, and decreased counts as -1. 
-- Varify that the results are ordered by the Product and Date.

CREATE VIEW vProductInventoriesWithPreviouMonthCountsWithKPIs AS 
  SELECT 
    TOP 100000
	 [Product Name] 
	,[Inventory Date] 
	,[Inventory Count] 
    ,[Previous Month Count] 
	,[KPI] = CASE  
	  WHEN [Inventory Count] > [Previous Month Count] THEN 1
	  WHEN [Inventory Count] = [Previous Month Count] THEN 0
      WHEN [Inventory Count] < [Previous Month Count] THEN -1
	 END
  FROM vProductInventoriesWithPreviouMonthCounts
  ORDER BY [Product Name], MONTH([Inventory Date]), YEAR([INVENTORY Date]);
GO 

-- Important: This new view must use your vProductInventoriesWithPreviousMonthCounts view!
-- Check that it works: 
SELECT * FROM vProductInventoriesWithPreviouMonthCountsWithKPIs;
GO

-- Question 8 (25% of pts): 
-- CREATE a User Defined Function (UDF) called fProductInventoriesWithPreviousMonthCountsWithKPIs.
-- Show columns for the Product names, Inventory Dates, Inventory Count, the Previous Month Count. 
-- The Previous Month Count is a KPI. The result can show only KPIs with a value of either 1, 0, or -1. 
-- Display months with increased counts as 1, same counts as 0, and decreased counts as -1. 
-- The function must use the ProductInventoriesWithPreviousMonthCountsWithKPIs view.
-- Varify that the results are ordered by the Product and Date.

CREATE FUNCTION fProductInventoriesWithPreviousMonthCountsWithKPIs(@Value INT)
  RETURNS TABLE 
  AS 
	RETURN 
	(SELECT 
	  	 [Product Name] 
	    ,[Inventory Date] 
	    ,[Inventory Count] 
        ,[Previous Month Count]
		,[KPI] 
	 FROM vProductInventoriesWithPreviouMonthCountsWithKPIs
	 WHERE [KPI] = @Value);
GO 

/* Check that it works:
Select * From fProductInventoriesWithPreviousMonthCountsWithKPIs(1);
Select * From fProductInventoriesWithPreviousMonthCountsWithKPIs(0);
Select * From fProductInventoriesWithPreviousMonthCountsWithKPIs(-1);
*/
GO
```
#### Results of Question 8

                        
![Results of Question 8](https://github.com/ronklnr/DBFoundations-Module07/blob/main/Results%20Question8.png "Results of Question 8")  
#### Figure 1. The results of Question 8
