---
layout: post
title: "Working with SQL queries"
date: 2016-12-15 08:07:00
categories: blogging
tags: sql
comments: true
analytics: true
---

In many of my projects I extensively use simple sql queries to work with data. Many of these queries are related to grouping the data, joining the tables or finding the top n results.

In this [repository](https://github.com/ksivam/sqlQueries) blog, I am going to present few simple customer/salesperson tables and write some sql queries show casing the usage of group by, join clause.

<br>

### Salesperson  

SalespersonID | Name | Age | Salary
------------- | ---- | --- | ------
1 | Alice | 61 | 140000
2 | Bob | 34 | 44000
6 | Chris | 34 | 40000
8 | Derek | 41 | 52000
11 | Emmit | 57 | 115000
16 | Fred | 38 | 38000

###  Customer  

CustomerID | Name
---------- | ----
4 | George
6 | Harry
7 | Ingrid
11 | Jerry

### Orders

OrderID | OrderDate | CustomerID | SalespersonID | NumberOfUnits | CostOfUnit
------- | --------- | ----------- | ------------ | ------------- | ----------
3 | 17/01/2013 | 4 | 2 | 4 | 400
6 | 07/02/2013 | 4 | 1 | 1 | 600
10 | 04/03/2013 | 7 | 6 | 2 | 300
17 | 15/03/2013 | 6 | 1 | 5 | 300
25 | 19/04/2013 | 11 | 11 | 7 | 300
34 | 22/04/2013 | 11 | 11 | 100 | 26
57 | 12/07/2013 | 7 | 11 | 14 | 11

**Return the names of all salespeople that
have an order with George**

```sql
--Return the names of all salespeople that have an order with George

select t3.Name as 'Name of sales people that have an order with George' from
(
-- get the customer id of George.
select CustomerID
from Customer
where UPPER(Name)=UPPER('George')
) as t1

inner join

(
-- get the sales person id that have an order with George.
select SalespersonID, CustomerID
from Orders
group by SalespersonID, CustomerID
) as t2
on t1.CustomerID=t2.CustomerID

inner join

(
-- get the sales person name that have an order with George.
select Name, SalespersonID
from Salesperson
group by Name, SalespersonID
) as t3
on t2.SalespersonID=t3.SalespersonID
```

**Return the names of all salespeople that do
not have any order with George**

```sql
-- Return the names of all salespeople that do not have any order with George

select Name as 'Name of sales people that donot have any order with George'
from Salesperson
where SalespersonID
-- get the sales person name that dont have any order with Gorge.
NOT IN
(
-- get the sales person id that have an order with George.
select SalespersonID
from Orders
where CustomerID
IN (
-- get the customer id of George.
select CustomerID
from Customer
where UPPER(Name)=UPPER('George')
)
)
```

**Return the names of salespeople that have
2 or more orders**

```sql
--Return the names of salespeople that have 2 or more orders.
select t2.Name as 'sales people that have 2 or more orders' from
(
--aggregate sales person id to get the number of orders per sales person.
select SalespersonID, COUNT(*) as NumOfOrders
from Orders
group by SalespersonID
) as t1

inner join

(
--get the sales person name who have more than 1 order (ie. 2 or more orders)
select Name, SalespersonID
from Salesperson
group by Name, SalespersonID
) as t2
on t1.SalespersonID=t2.SalespersonID and t1.NumOfOrders > 1
```

**Return the name of the salesperson with
the 3rd highest salary.**

```sql
--Return the name of the salesperson with the 3rd highest salary.

-- get the last sales person in the top 3 highest saleried sales person.
select top 1 t1.Name as 'sales person with 3rd highest salary' from
(
-- get the top 3 highest saleried sales person.
select top 3 Name, Salary
from Salesperson
order by Salary desc
) as t1
order by t1.Salary asc
```

**Create a new roll­up table BigOrders(where
columns are CustomerID,
TotalOrderValue), and insert into that table
customers whose total Amount across all
orders is greater than 1000**

```sql
--Create a new rollup table BigOrders(where columns are CustomerID, TotalOrderValue), and insert into that table
--customers whose total Amount across all orders is greater than 1000

-- insert the customers whose total order value is greater than 100
select t1.CustomerID, t1.TotalOrderValue into BigOrders from
(
--aggregate the order value for each customer
select CustomerID, SUM(CostOfUnit) as TotalOrderValue
from Orders
group by CustomerID
) as t1
where t1.TotalOrderValue > 1000
```

**Return the total Amount of orders for each
month, ordered by year, then month (both
in descending order)**

```sql
--Return the total Amount of orders for each month, ordered by year, then month (both in descending order)

select YEAR(OrderDate) as Year,  MONTH(OrderDate) as Month, SUM(CostOfUnit) as TotalAmountOfOrders
from Orders
group by YEAR(OrderDate), MONTH(OrderDate)
order by TotalAmountOfOrders desc
```

The above sql queries clearly demonstrate the usage of group by, order by, inner join and top clause.

Also, i have found the below venn diagram to be super useful while constructing the join clause.
![SQL join venn diagram](/assets/images/sql_joins_venn.jpg)
