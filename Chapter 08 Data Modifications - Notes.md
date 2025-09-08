# 8.1 Inserting Data

First, create **dbo.Orders** table

```
DROP TABLE IF EXISTS dbo.Orders;

CREATE TABLE dbo.Orders
(
  orderid   INT         NOT NULL
    CONSTRAINT PK_Orders PRIMARY KEY,
  orderdate DATE        NOT NULL
    CONSTRAINT DFT_orderdate DEFAULT(SYSDATETIME()),
  empid     INT         NOT NULL,
  custid    VARCHAR(10) NOT NULL
);
```

## 8.1.1 INSERT INTO ... VALUES ... statement

### 8.1.1.1 Insert a Single row

```
insert into dbo.orders(orderid, orderdate, empid, custid)
values(10001, '20160212', 3, 'A');
```

lets see what has been inserted?

```
select * from orders
```

<img width="247" height="45" alt="image" src="https://github.com/user-attachments/assets/abe8ea9d-0423-4a4d-9ad6-c1a5d74668e6" />


if you do not specify a value, the system will use **default value**, if no default value is specified in the table creation, the system will use a NULL.


```
insert into orders(orderid, empid, custid)
values(10002, 5, 'B')
```

this insert used system default date value.

<img width="244" height="66" alt="image" src="https://github.com/user-attachments/assets/26c82e03-fa6a-4082-be43-0a301c8751f9" />



### 8.1.1.2 Insert Multiple rows

```
insert into orders
(orderid, orderdate, empid, custid)
values
	(10003, '20160213', 4, 'B'),
	(10004, '20160214', 1, 'A'),
	(10005, '20160213', 1, 'C'),
	(10006, '20160215', 3, 'C');
```


<img width="248" height="141" alt="image" src="https://github.com/user-attachments/assets/f8b23bb3-2d36-43cd-9292-185005e822f9" />



### 8.1.1.3 Table-value Constructor

you can use the `VALUES` clause as a table-value constructor

```
select *
from (
		values
			(10003, '20160213', 4, 'B'),
			(10004, '20160214', 1, 'A'),
			(10005, '20160213', 1, 'C'),
			(10006, '20160215', 3, 'C'))
		as O(orderid, orderdate, empid, custid);
```

<img width="241" height="104" alt="image" src="https://github.com/user-attachments/assets/46d61eeb-3f9b-4db8-96bf-0a7860fefde9" />








---


## 8.1.2 INSERT INTO ...SELECT ... FROM ... statement

```
select * from Sales.Orders
where shipcountry = 'UK';
```

we get 56 rows.

```
insert into dbo.Orders(orderid, orderdate, empid, custid)
select orderid, orderdate, empid, custid
from Sales.Orders
where shipcountry = 'UK';
```

---


## 8.1.3 INSERT INTO ... EXEC... statement

First create a restored procedure

```
Drop proc if exists sales.GetOrders;
Go

Create proc sales.GetOrders
	@country as nvarchar(40)
as
Select orderid, orderdate, empid, custid
from Sales.Orders
Where shipcountry = @country
```

Test out the Stored Proc

```
EXEC sales.getorders @country = N'France'
```

77 rows returned

```
insert into dbo.orders(orderid, orderdate, empid, custid)
	exec sales.GetOrders @country = N'France'
```


## 8.1.4 SELECT ... INTO ... FROM ...

* this is non ANSI
* can only be used to create new tables



```
drop table if exists dbo.orders;

select orderid, orderdate, empid, custid
into dbo.orders
from sales.orders
```



---



### SELECT INTO FROM with Set operation

```
select country, region, city
from sales.customers
except
select country, region, city
from HR.Employees
```
<img width="308" height="310" alt="image" src="https://github.com/user-attachments/assets/1e29858f-64bd-4755-930e-b04e29f270b5" />

66 rows returned


put the INTO clause in the first select statement


```
drop table if exists dbo.locations

select country, region, city
INTO dbo.locations
from sales.customers
except
select country, region, city
from HR.Employees
```

```
select * from locations
```


*66 rows returned*



## 8.1.5 BULK INSERT 

used to insert a file into SQL Server table.


```
delete from orders

bulk insert dbo.orders from 'C:\Users\benja\OneDrive\Onedrive\Self-Studies\1058 - Ben-Gan, T-SQL Fundamentals 3e\Chapter 08. Data modification\orders.txt'
with
	(
		datafiletype = 'char',
		fieldterminator = ',',
		rowterminator = '\n'

	);

select * from orders
```


<img width="246" height="222" alt="image" src="https://github.com/user-attachments/assets/70ab593a-733f-4d14-9b08-b81a246ab6e7" />

10 rows returned


## 8.1.6 Identity Property and the sequence object


### 8.1.6.1 Identity
**Identity** is used to generate *Surrogate Keys*


```
drop table if exists dbo.T1;

create table dbo.T1
(
	keycol int not null identity(1,1)
		constraint PK_T1 primary key,
	datacol varchar(10) not null
		constraint chk_T1_dotacol check(datacol like '[ABCDEFGHIJKLMNOPQRSTUVWXYZ]%')
);
```

we must ignore the Identity column when inserting data.

```
 insert into dbo.T1(datacol)
 values	('AAA'),
		('BBB'),
		('CCC');
```

<img width="134" height="82" alt="image" src="https://github.com/user-attachments/assets/3d140965-c1a1-4ed8-b025-0fd679d400b7" />

To select an identiy column, use the *Keycol* or generic *$identity*

```
select $identity from T1;
```

<img width="84" height="83" alt="image" src="https://github.com/user-attachments/assets/656b8f15-d0fc-4b83-be5f-626477dd12da" />

`SCOPE_IDENTITY` returns the last identity value generated in the current scope.

```
 declare @new_key as int;

 insert into dbo.T1(datacol) values('AAAAA');

 set @new_key = SCOPE_IDENTITY();

 select @new_key as new_key
```

<img width="99" height="44" alt="image" src="https://github.com/user-attachments/assets/92363b6a-6dc7-48e0-a32f-b06537d2c596" />


```
select
SCOPE_IDENTITY() as [Scope_identity]
, @@IDENTITY as [@@idenity]
, IDENT_CURRENT('dbo.T1') as [Ident_current];
```

retrieve the last identity value regardless which session, use `IDENT_CURRENT`

<img width="269" height="45" alt="image" src="https://github.com/user-attachments/assets/e72fe89e-b655-4129-887f-96d52348e74e" />



#### insert explicit Identity value

We will need to turn on `SET IDENTITY_INSERT`

```
SET IDENTITY_INSERT T1 On;
Insert into T1(keycol, datacol) values(18, 'YYY');
SET IDENTITY_INSERT T1 Off;
```

#### Reset Identity value

use `DBCC CHECKIDENT` command to reseed, ie change the value of identity back to 0








### 8.1.6.2 Sequence

The default min and max values of a sequence is the min and max value supported by the data type. Therefore it is important to explicit define the min value.

```
create sequence dbo.SeqOrderIDs as int
	minvalue 1
	cycle;
```
To alter the sequence property, use `ALTER`

```
alter sequence dbo.SeqOrderIDs
	No cycle;
```

```
select next value for dbo.SeqOrderIDs
```

<img width="139" height="43" alt="image" src="https://github.com/user-attachments/assets/d8d0e57f-ae8c-4507-aeb6-dc32b58352c0" />


create a new table called **T1**.


```
drop table if exists dbo.T1

create table dbo.T1
(
	keycol int not null
		constraint pk_T1 primary key, 
	datacol varchar(10) not null
)
```
#### Inserting new sequence value through a variable

```
declare @neworderid as int = next value for dbo.SeqOrderIDs;
insert into dbo.T1(keycol, datacol) values(@neworderid, 'a');

select * from dbo.T1
```

<img width="136" height="45" alt="image" src="https://github.com/user-attachments/assets/7665c03b-6bce-45da-a1f7-b6262a600428" />

#### Inserting new sequence value directly

```
INSERT INTO dbo.T1(keycol, datacol)
	values(NEXT VALUE FOR dbo.SeqOrderIDs, 'b');

select * from dbo.T1
```

<img width="136" height="63" alt="image" src="https://github.com/user-attachments/assets/938fe330-ed9f-4445-a5cb-70a8b688d525" />

#### update column with Sequence

```
UPDATE dbo.T1
	SET keycol = NEXT VALUE FOR dbo.SeqOrderIDs;

select * from dbo.T1
```

<img width="144" height="62" alt="image" src="https://github.com/user-attachments/assets/3d8c11d3-2b66-4cab-8bdb-37ac118ef1e8" />

#### to find the current value of a Sequence.

```
select current_value
from sys.sequences
where OBJECT_ID = OBJECT_ID(N'dbo.SeqOrderIDs')
```

#### multirow INSERT using OVER()

```
insert into dbo.T1(keycol, datacol)
	select
		next value for dbo.SeqOrderIDs over(order by hiredate),
		left(firstname, 1) + left(lastname, 1)
	from hr.Employees

select * from dbo.T1;
```

<img width="131" height="235" alt="image" src="https://github.com/user-attachments/assets/6b380623-e5ce-4335-b57f-c21da13883da" />


#### adding sequence value as part of the Table Constraint

adding the constraint

```
ALTER TABLE dbo.T1
	Add constraint DFT_T1_Keycol
	 Default (next value for dbo.SeqOrderIDs)
	 for keycol;
```

inserting the data

```
insert into dbo.T1(datacol) values('C');

select * from T1;
```

<img width="134" height="258" alt="image" src="https://github.com/user-attachments/assets/d0d9ac87-fccd-49a2-b0d9-ab7dc16d1db0" />


#### sp_sequence_get_range

```
declare @first as sql_variant;

exec sys.sp_sequence_get_range
	@sequence_name = 'dbo.SeqOrderIDs',
	@range_size = 1000000,
	@range_first_value = @first output;

select @first
```

#### clean up

```
drop table if exists dbo.T1;
drop sequence if exists dbo.SeqOrderIDs;
```

# 8.2 Deleting Data

```
DROP TABLE IF EXISTS dbo.Orders, dbo.Customers;

CREATE TABLE dbo.Customers
(
  custid       INT          NOT NULL,
  companyname  NVARCHAR(40) NOT NULL,
  contactname  NVARCHAR(30) NOT NULL,
  contacttitle NVARCHAR(30) NOT NULL,
  address      NVARCHAR(60) NOT NULL,
  city         NVARCHAR(15) NOT NULL,
  region       NVARCHAR(15) NULL,
  postalcode   NVARCHAR(10) NULL,
  country      NVARCHAR(15) NOT NULL,
  phone        NVARCHAR(24) NOT NULL,
  fax          NVARCHAR(24) NULL,
  CONSTRAINT PK_Customers PRIMARY KEY(custid)
);

CREATE TABLE dbo.Orders
(
  orderid        INT          NOT NULL,
  custid         INT          NULL,
  empid          INT          NOT NULL,
  orderdate      DATE         NOT NULL,
  requireddate   DATE         NOT NULL,
  shippeddate    DATE         NULL,
  shipperid      INT          NOT NULL,
  freight        MONEY        NOT NULL
    CONSTRAINT DFT_Orders_freight DEFAULT(0),
  shipname       NVARCHAR(40) NOT NULL,
  shipaddress    NVARCHAR(60) NOT NULL,
  shipcity       NVARCHAR(15) NOT NULL,
  shipregion     NVARCHAR(15) NULL,
  shippostalcode NVARCHAR(10) NULL,
  shipcountry    NVARCHAR(15) NOT NULL,
  CONSTRAINT PK_Orders PRIMARY KEY(orderid),
  CONSTRAINT FK_Orders_Customers FOREIGN KEY(custid)
    REFERENCES dbo.Customers(custid)
);
GO

INSERT INTO dbo.Customers SELECT * FROM Sales.Customers;
INSERT INTO dbo.Orders SELECT * FROM Sales.Orders;
```

## 8.2.1 DELETE Statement

```
delete from dbo.orders
where orderdate < '20150101';
```

<img width="543" height="75" alt="image" src="https://github.com/user-attachments/assets/0ee1445d-1ff9-48af-ad60-03e882594a63" />

note: `DELETE` is expensive because it is fully logged.

```
select * from orders
```

678 rows returned (originally there was 830 rows, 152 rows were deleted)

## 8.2.2 TRUNCATE Statement

```
truncate table dbo.T1
```

truncate tables with partitions

```
truncate table dbo.T1 with (partition(1,3,5 to 10))
```

## 8.2.3 DELETE Based on a join (non-ANSI)

Used to delete rows from one table used on filter in another table, avoid in favour of using a subquery.

```
delete from O
from dbo.Orders as O
	inner join dbo.customers as C
				on O.custid = C.custid
	where C.country = N'USA';
```

(122 rows affected)


#### Subquery equivalent

```
delete from dbo.orders
where exists
	(select * 
		from dbo.customers as c
		where orders.custid = c.custid
		and c.country = 'USA')
```

(122 rows affected)

---

# 8.3 Updating Data (page 266)

```
DROP TABLE IF EXISTS dbo.OrderDetails, dbo.Orders;

CREATE TABLE dbo.Orders
(
  orderid        INT          NOT NULL,
  custid         INT          NULL,
  empid          INT          NOT NULL,
  orderdate      DATE         NOT NULL,
  requireddate   DATE         NOT NULL,
  shippeddate    DATE         NULL,
  shipperid      INT          NOT NULL,
  freight        MONEY        NOT NULL
    CONSTRAINT DFT_Orders_freight DEFAULT(0),
  shipname       NVARCHAR(40) NOT NULL,
  shipaddress    NVARCHAR(60) NOT NULL,
  shipcity       NVARCHAR(15) NOT NULL,
  shipregion     NVARCHAR(15) NULL,
  shippostalcode NVARCHAR(10) NULL,
  shipcountry    NVARCHAR(15) NOT NULL,
  CONSTRAINT PK_Orders PRIMARY KEY(orderid)
);

CREATE TABLE dbo.OrderDetails
(
  orderid   INT           NOT NULL,
  productid INT           NOT NULL,
  unitprice MONEY         NOT NULL
    CONSTRAINT DFT_OrderDetails_unitprice DEFAULT(0),
  qty       SMALLINT      NOT NULL
    CONSTRAINT DFT_OrderDetails_qty DEFAULT(1),
  discount  NUMERIC(4, 3) NOT NULL
    CONSTRAINT DFT_OrderDetails_discount DEFAULT(0),
  CONSTRAINT PK_OrderDetails PRIMARY KEY(orderid, productid),
  CONSTRAINT FK_OrderDetails_Orders FOREIGN KEY(orderid)
    REFERENCES dbo.Orders(orderid),
  CONSTRAINT CHK_discount  CHECK (discount BETWEEN 0 AND 1),
  CONSTRAINT CHK_qty  CHECK (qty > 0),
  CONSTRAINT CHK_unitprice CHECK (unitprice >= 0)
);
GO

INSERT INTO dbo.Orders SELECT * FROM Sales.Orders;
INSERT INTO dbo.OrderDetails SELECT * FROM Sales.OrderDetails;
```

## 8.3.1 UPDATE statement

```
select discount from orderdetails
where productid = 51
```

<img width="91" height="561" alt="image" src="https://github.com/user-attachments/assets/aa67ceef-36b3-4bb7-be3b-8b5b257cc890" />


```
update OrderDetails
set discount = discount + 0.05
where productid = 51;
```

_(39 rows affected)_

```
select discount from orderdetails
where productid = 51
```

<img width="92" height="674" alt="image" src="https://github.com/user-attachments/assets/4d6b5624-ec4b-49e9-b90f-5952b63748d7" />

We can use `OUTPUT` clause to see the changes


### using Compound Assignment operators 

We can use `SET discount += 0.05` instead of `SET discount = discount + 0.05`

```
update OrderDetails
SET discount += 0.05
where productid = 51;
```



### All-at-once Operation

```
drop table if exists t1
create table t1(col1 int, col2 int)
insert into t1 values (100,0)

select * from t1
```

<img width="116" height="54" alt="image" src="https://github.com/user-attachments/assets/32b9113c-18c0-4598-86f2-72c3d94b9c33" />


```
update t1
set col1 = col1 + 10, col2 = col1 + 10;
select * from t1
```
<img width="115" height="46" alt="image" src="https://github.com/user-attachments/assets/91cae4e5-f730-4cea-87fa-adfce695426a" />


reset t1

```
update t1
set col1=col2, col2=col1;
select * from t1;
```

<img width="110" height="50" alt="image" src="https://github.com/user-attachments/assets/f51d8cb4-6d20-4f91-9ec4-a2c608c44724" />

## 8.3.2 UPDATE based on a join

### Remember this is non-ANSI

```
update OD
	set discount +=0.05
from OrderDetails as OD
	inner join Orders as O
	on OD.orderid = O.orderid
Where O.custid = 1
```

### The standard version

```
Update dbo.OrderDetails
	set discount += 0.05
where exists
	(select * from dbo.Orders as O
	where O.orderid = OrderDetails.orderid
	and O.custid = 1);
```





## 8.3.3 Assignment UPDATE


```
drop table if exists dbo.MySequences;

create table dbo.MySequences
(
	id Varchar(10) not null
		constraint PK_mySequences primary key(id),
	val int not null
);
insert into dbo.MySequences Values('SEQ1', 0);
```







# 8.4 Merging Data


```
-- Listing 8-2 Code that Creates and Populates Customers and CustomersStage
DROP TABLE IF EXISTS dbo.Customers, dbo.CustomersStage;
GO

CREATE TABLE dbo.Customers
(
  custid      INT         NOT NULL,
  companyname VARCHAR(25) NOT NULL,
  phone       VARCHAR(20) NOT NULL,
  address     VARCHAR(50) NOT NULL,
  CONSTRAINT PK_Customers PRIMARY KEY(custid)
);

INSERT INTO dbo.Customers(custid, companyname, phone, address)
VALUES
  (1, 'cust 1', '(111) 111-1111', 'address 1'),
  (2, 'cust 2', '(222) 222-2222', 'address 2'),
  (3, 'cust 3', '(333) 333-3333', 'address 3'),
  (4, 'cust 4', '(444) 444-4444', 'address 4'),
  (5, 'cust 5', '(555) 555-5555', 'address 5');

CREATE TABLE dbo.CustomersStage
(
  custid      INT         NOT NULL,
  companyname VARCHAR(25) NOT NULL,
  phone       VARCHAR(20) NOT NULL,
  address     VARCHAR(50) NOT NULL,
  CONSTRAINT PK_CustomersStage PRIMARY KEY(custid)
);

INSERT INTO dbo.CustomersStage(custid, companyname, phone, address)
VALUES
  (2, 'AAAAA', '(222) 222-2222', 'address 2'),
  (3, 'cust 3', '(333) 333-3333', 'address 3'),
  (5, 'BBBBB', 'CCCCC', 'DDDDD'),
  (6, 'cust 6 (new)', '(666) 666-6666', 'address 6'),
  (7, 'cust 7 (new)', '(777) 777-7777', 'address 7');
```



**Customers** table
<img width="314" height="122" alt="image" src="https://github.com/user-attachments/assets/4c03186b-ed79-4547-b14d-eac5422f1028" />



**CustomersStage** table

<img width="324" height="125" alt="image" src="https://github.com/user-attachments/assets/3fa897a4-c69f-4159-8b85-c679d9ddf959" />


### Add nonexistent customers and update existing ones

```
merge into customers as TGT
using CustomersStage as SRC
	on TGT.custid = SRC.custid
when matched then
	update set 
			tgt.companyname = src.companyname,
			tgt.phone = src.phone,
			tgt.address = src.address
when not matched then
	insert (custid, companyname, phone, address)
	values (src.custid, src.companyname, src.phone, src.address);
```

_(5 rows affected)_

<img width="314" height="122" alt="image" src="https://github.com/user-attachments/assets/4c03186b-ed79-4547-b14d-eac5422f1028" />

compare against the original **customer** table

<img width="309" height="159" alt="image" src="https://github.com/user-attachments/assets/d4e7cc70-43be-49dc-a220-83df7282f7b4" />



### WHEN NOT MATCHED BY SOURCE

```
merge into customers as TGT
using CustomersStage as SRC
	on TGT.custid = SRC.custid
when matched then
	update set 
			tgt.companyname = src.companyname,
			tgt.phone = src.phone,
			tgt.address = src.address
when not matched then
	insert (custid, companyname, phone, address)
	values (src.custid, src.companyname, src.phone, src.address)
WHEN NOT MATCHED BY SOURCE THEN
	DELETE;
```

<img width="313" height="122" alt="image" src="https://github.com/user-attachments/assets/b01ff90d-9f4e-4587-8383-569720acf6a0" />


### apply update if at least one column value is different

Use `AND` in the `WHEN MATCHED` clause.


```
merge into customers as TGT
using CustomersStage as SRC
	on TGT.custid = SRC.custid
WHEN MATCHED AND
			(tgt.companyname <> src.companyname
			 OR tgt.phone <> src.phone
			 OR tgt.address <> src.address) THEN
	update set 
				tgt.companyname = src.companyname,
				tgt.phone = src.phone,
				tgt.address = src.address

WHEN NOT MATCHED THEN
	insert (custid, companyname, phone, address)
	values (src.custid, src.companyname, src.phone, src.address);
```


# 8.5 Modifying data through table expressions (page 276)



























































































































































































