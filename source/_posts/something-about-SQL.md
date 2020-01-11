---
title: SQL重难点总结
comments: true
date: 2018-04-11 23:04:17
updated: 2018-04-11 23:04:17
tags: [SQL]
categories: [Database]
permalink:
---
最近在看数据库的书籍，对数据库的操作，尤其是SQL语言更加熟悉了，概念方面理顺了许多。对以前比较陌生的东西也不再有畏惧感。所以写这篇博客把重难点的部分一并总结一下。

以下的SQL基于**SQL必知必会**书上的五张表:Vendors,Products,Customers,Orders,OrderItems.

# 分组数据
## 数据分组
如下我们可以计算供应商DLL01提供的产品数目：
``` sql
SELECT COUNT(*) AS num_prods 
FROM Products 
WHERE vend_id = "DLL01";
```
那如果我们需要分别计算每一个供应商的提供的产品的数量呢?这个时候就是分组大显身手的时候了。**使用分组可以将数据分为多个逻辑组，对每个组进行聚集计算**。如下:
``` sql
SELECT vend_id,COUNT(*) AS num_prods 
FROM Products 
GROUP BY vend_id;
```
上面的SELECT语句指定了两个列：vend\_id包含产品供应商的ID,num\_prods为计算字段(用COUNT(*)函数建立)。`GROUP BY`子句指示DBMS按照vend\_id排序并分组数据。这就会对每个vend\_id而不是整个表计算一次num\_prods.
## 过滤分组
除了可以使用`GROUP BY`进行分组之外，还可以过滤分组，决定那些分组需要，那些分组可以滤去。过滤分组通过HAVING子句进行。例如下面过滤出产品数大于等于2的分组:
``` sql
SELECT vend_id,COUNT(*) AS num_prods 
FROM Products 
GROUP BY vend_id 
HAVING COUNT(*) >= 2;
```
上面的语句中只有产品数目大于等于2的分组才能够被选择出来。
## HAVING VS WHERE
众所周知，`WHERE`子句具有过滤的作用，但是`WHERE`子句指定的是过滤而不是分组，事实上`WHERE`没有分组的概念。`HAVING`非常类似于`WHERE`,目前绝大多数的`WHERE`子句都可以用`HAVING`来替代，唯一的差别是，`WHERE`过滤行，而`HAVING`过滤分组。另一种理解方式是：`WHERE`在数据分组前进行过滤，`HAVING`在数据分组后进行过滤。`WHERE`排除的行不包括在分组中。这可能会改变计算值，从而影响`HAVING`子句中基于这些值过滤掉的分组。`HAVING`与`WHERE`非常类似，如果不指定`GROUP BY`,则大多数的DBMS会同等的对待他们。不过我们自己还是需要区分这一点的，使用**HAVING时应该结合GROUP BY子句，而WHERE子句用于标准的行级过滤**。

## SELECT子句顺序
| 子句 |　说明　|　是否必须使用　|
| :----: | :-----: | :----------: |
| SELECT | 要返回的列或表达式 | 是　|
| FROM   | 从中检索数据的表 |　仅在从表选择数据时使用　|
| WHERE   | 行级过滤 |　否　|
| GROUP BY   | 分组说明 |　仅在按组计算聚集时使用　|
| HAVING   | 组级过滤 |　否　|
| ORDER BY   | 输出排序顺序 |　否　|

# 联结表
SQL最强大的功能之一就是能在数据查询的执行中联结(join)表，联结是利用SQL的SELECT能执行的最重要的操作。为什么要使用联结操作呢？将数据分解为多个表能更有效的存储，更方便的处理，并且可伸缩性更好，这是关系型数据库的最大的特点。但是这些好处是有代价的。我们需要通过联结多个表返回一组输出，联结在运行时关联表中的正确的行。
## 内联结
``` sql
SELECT vend_name,prod_name,prod_price
FROM Vendors,Products
WHERE Vendors.vend_id = Products.vend_id;
```
如上，我们用`WHERE`子句将Products表的外键vend\_id与它所引用的Vendors表的主键vend\_id关联起来,这样就建立了这两张表的联结。上面的联结的大致执行过程是这样的：首先对于Vendors表中的每一个元组，都去比对Products表中的每一个元组，如果满足Vendors.vend\_id = Products.vend\_id,就从这两个元组中提取出指定的列，然后放到输出中，接着就去取Vendors表中的下一个元组，重复上面的过程。Vendors表中的所有元组执行完毕。这里有一个问题需要注意：对于Vendors表中的一个元组，如果在Products表中找不到满足Vendors.vend\_id = Products.vend\_id的元组(也就是该供应商没有提供产品)，这时候Vendors表中的这个元组放不放到输出中呢？对于上面的SQL，这样的元组不会放到输出中。

上面使用的联结称为**等值联结(equijoin)**，它基于两个表之间的相等测试。这种联结也称为**内连接(inner join)**。其实，可以对这种联结使用稍微不同的语法，明确指定联结类型，就像下面这样:
``` sql
SELECT vend_name, prod_name,prod_price
FROM Vendors INNER JOIN Products
ON Vendors.vend_id = Products.vend_id;
```
这条SQL语句和前面的那条语句所起的作用是等价的。这里，两个表之间的关系是以`INNER JOIN`指定的部分`FROM`子句，在使用这种语法时，联结条件用特定的`ON`子句而不是`WHERE`子句给出，传递给`ON`的实际条件与传递给`WHERE`的相同。

## 联结多个表
SQL不限制一条SELECT语句中可以联结的表的数目，创建联结的基本规则也相同，首先列出所有表，然后定义表之间的关系，就像下面这样：
``` sql
SELECT prod_name,vend_name,prod_price,quantity
FROM OrderItems,Products,Vendors
WHERE Products.vend_id = Vendors.vend_id
AND OrderItems.prod_id = Products.prod_id
AND order_num = 20007;
```

上面我们使用的只是内联结(也叫等值联结)的简单联结，下面我们看其他联结：自联结(self-join)、外联结(outer join).
## 自联结
使用表别名能够在一个SELECT语句中不止一次的引用相同的表，我们可以使用利用表别名来完成同一个表的自联结。如下：
``` sql
SELECT c1.cust_id, c1.cust_name,c1.cust_contact
FROM Customers AS c1,Customers AS c2
WHERE c1.cust_name = c2.cust_name
AND c2.cust_contact = "Jim Jones";
```
这条语句检索出的就是和Jim Jones在同一家公司工作的的雇员的信息。

## 外联结
内连接是将一个表中的行与另一个表中的行相关联，但有时候需要关联没有关联行的那些行。例如可能需要使用联结完成以下工作：
－　对每一个顾客下的订单数进行计数，包括那些至今尚未下订单的顾客；
－　列出所有产品以及订购数量，包括没有人订购的产品；
－　计算平均销售规模，包括那些至今尚未下订单的顾客。
在上述例子中，联结包含了那些在相关表中没有关联行的行。这种联结称为**外联结**。如下的例子：
``` sql
SELECT Customers.cust_id,Orders.order_num
FROM Customers LEFT OUTER JOIN Orders
ON Customers.cust_id = Orders.cust_id;
```
这条SELECT语句使用了关键字`OUTER JOIN`来指定了联结类型。与内连接关联两个表中的行不同，外联结还包括没有关联的行。在使用`OUTER JOIN`语法时，必须使用RIGHT或者LEFT关键字指定包括其所有行的表(`RIGHT`指出的是`OUTER JOIN`右边的表，而`LEFT`指出的是`OUTER JOIN`左边的表)，上面的例子使用`LEFT OUTER JOIN`从`FROM`子句左边的表(Customers)中选择所有行。为了从右边的表中选择所有的行，需要使用`RIGHT OUTER JOIN`,如下：
``` sql
SELECT Customers.cust_id,Orders.order_num
FROM Customers　RIGHT OUTER JOIN Orders
ON Customers.cust_id = Orders.cust_id;
```
## 使用带聚集函数的联结
如下，按照顾客的id分组，查询每个顾客的订单数。
``` sql
SELECT Customers.cust_id,
    COUNT(Orders.order_num) AS num_ord
FROM Customers INNER JOIN Orders 
ON Customers.cust_id = Orders.cust_id
GROUP BY Customers.cust_id;
```
这里使用的是内联结，所以不会包含没有下订单的顾客分组。如果要包含他们就需要使用`LEFT OUTER JOIN`。
