-강사님, 그리고 담당자님. 평소 제가 할 수 있는 것 보다 훨씬 더 그 이상으로 최선을 다해 통합실습 및 테스트에 임했습니다. 최대한 적을 수 있는 것을 다 적고, 다 하고 모든 걸 쏟아부었습니다. 더 이상 못 할 것 같습니다 정말...ㅠ 제발 fail 시키지 말아주세요 제발 ㅠㅠ

###1

-Create Database
```
create database training;
use training;
```
-Make Table in Hive
```
create external table account (
id STRING, status STRING, amount INT, type STRING)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
LOCATION '/home/training/problem1'
```
-Excute Query
```
select a.id, a.type, a.status, a.amount,
(b.average - a.amount) as difference
from account a
join
(select type, avg(amount) as average from account where status = "Active"
group by type) b
on (a.type = b.type)
where status = "Active";
```


###2

-Instructions
```
Create an employee table in the metastore that contains the employee records stored in HDFS.
[Data Description]
All of the employee records are stored in the /user/training/problem2/data/employee/ HDFS directory in Parquet
file format. The files contain the following columns and types:
```
-Output Requirements
1. The table you create should be named solution and stored in the problem2 database
2. The table must point to the existing data in HDFS

-Make Table
```
CREATE EXTERNAL TABLE solution (
id int, fname string, lname string, address string, city string, state string, zip string, birthday string,
hireday string )
stored as parquet
location “/user/training/problem2/data/employee/“
```

###3

-Instructions
```
Generate a table that contains all customers who have negative account balances.
[Data Description]
The customer records are stored in the customer table in the problem3 database. The account records are
stored in the account table in the problem3 database. The account records contain a field called amount that
may be negative. The custid field is a foreign key into the customer table.
```
-Output Requirements
1. Create a metastore table named solution stored in the problem3 database that contains the customer
records that match the criteria
2. Each solution record should contain the Customer ID, first name, last name, and home phone of the
customer
3. Maintain the same column names and datatypes in the new table as the fields from the customer table

-Make Table
```
create external table solution3
( id STRING, fname string, lname string, hphone string)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
LOCATION '/user/training/problem3/customer'
```
-review
```
select * from solution;
```

###4

-Instructions
LoudAcre Mobile has merged with another company located in California. Each company has a list of customers
in different formats. Combine the two customer lists into a single dataset using an identical schema.

-Data Description
The original customer data exists in the HDFS directory /user/training/problem4/data/employee1/. It contains
expanded, nine-digit zip codes. The new files are in the HDFS directory
/user/training/problem4/data/employee2/. It contains last names before first names, both using all capital letters.

-Output Requirements
1. Combine these files into a single tab-delimited dataset and stored in the HDFS directory
/user/training/problem4/solution/
2. Each record should be in the following format:
3. Only include customers whose state is ‘CA’

-Excute on PIG
```
employee1 = LOAD '/user/training/problem4/data/employee1' AS (customerID:Int, fname:chararray,
lname:chararray, address:chararray, city:chararray, state:chararray, zip: chararray);
e1 = FOREACH employee1 GENERATE customerID, fname, lname, address, city, state,
SUBSTRING(zip,0,5);

employee2 = LOAD '/user/training/problem4/data/employee2' USING PigStorage(',') AS (customerID:Int,
junk:Int, lname:chararray, fname:chararray, address:chararray, city:chararray, state:chararray, zip:
chararray);

e2 = FOREACH employee2 GENERATE customerID, UCFIRST(LOWER(fname)),
UCFIRST(LOWER(lname)), address, city, state, zip;

both = UNION e1 , e2;

STORE both INTO '/user/training/problem4/solution/'
```

###5

-Instructions
The bank is making a Facebook group for the Palo Alto, CA branch. Generate a script that outputs the
customers and employees who live in Palo Alto, CA.

-Data Description
The employee records are stored in the employee metastore table in the problem5 database, The customer
records are stored in the customer metastore table in the problem5 database.

-Output Requirements
1. Write the report query in the local file /home/training/problem5/solution.sql
2. Executing the solution.sql script from the hive command-line tools should generate the report output
3. The report should contain first name, last name, city, and state with a tab as the delimiter between fields
4. Only output records that have a city value of ‘Palo Alto’ and state value of ‘CA’
5. Duplicate records (if any) should be included

-Make Table
```
create external table customer5 ( id int, fname string, lname string, address string, city string, state
string, zip string) ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t' LOCATION
"/home/training/problem5/customer";

create external table employee5 ( id int, fname string, lname string, address string, city string, state
string, zip string) ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t' LOCATION
"/home/training/problem5/employee";
```
-Excute Query
```
select fname, lname, city, state from customer5 where (city = "Palo Alto" AND state = "CA") UNION
ALL select fname, lname, city, state from employee5 where (city = "Palo Alto" AND state = "CA");
```

###6

-Instructions
There are privacy concerns about the employee data that is stored on the cluster. Your task is to remove any age
information from the employee data by creating a new table for the data analysts to query against.

-Data Description
All of the employee records are stored in the employee metastore table in the problem6 database.

-Output Requirements-
1. Create a new table named solution stored in the problem6 database with the same file format as the
employee table
2. Maintain the same column names and datatypes in the new table
3. The birthday field in the solution table should be truncated to only contain month/day instead of the
current month/day/year data that is in the employee table
```
create external table employee6 ( id int, fname string, lname string, address string, city string, state string,
zip string, birthday string) ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t' LOCATION
"/user/training/problem6/employee";

create table solution6 as select id, fname, lname, address, city, state, zip, substr(birthday,1,5) from
employee6;
```
-review
```
select * from solution6;
```
###7

-Instructions
Generate a report that contains all of the Seattle employee names in sorted order.

-Data Description
The employee records are stored in the employee table in the problem7 database.

[Output Requirements]
1. Write the report query in the local file /home/training/problem7/solution.sql
2. Executing the solution.sql script from the hive command-line tools should generate the report output
3. The employee names should be printed out as the first name, a space, and the last name
4. The output should be sorted by first name and then by last name and should only contain employees
whose city is ‘Seattle’
5. Duplicate names should be included (if any)

-Make Table
```
create external table employee7 ( id int, fname string, lname string, address string, city string, state
string, zip string, birthday string) ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
LOCATION "/home/training/problem7/employee";
```
-Excute Query
```
select concat(fname, " ", lname) as name
from employee7
where city = "Seattle"
order by name;
```

###8

-Instructions
Use Sqoop to export customer data from HDFS into a MySQL database table. Place the data in the solution
table in MySQL, which has been created and is currently empty.

-Data Description
The data files are in the HDFS directory /user/training/problem8/data/customer/. MySQL database information:
1. Installation: localhost
2. Database name: problem8 l Table name: solution
3. Username: cloudera
4. Password: cloudera

-Output Requirements-
1. Export all of the customer data from HDFS into the MySQL solution table
2. The solution table already exists in the MySQL database but currently has no rows
```
sqoop export
--connect jdbc:mysql://localhost/problem8
--username cloudera
--password cloudera
--table solution
--input-fields-terminated-by '\t'
--export-dir hdfs://localhost:8020/user/training/problem8/data/customer
```
-review
```
select * from solution;
```

###9

-Instructions
Your company is being acquired by another company. To prepare for this acquisition, update the customer
records to guarantee there will be no duplicate IDs with their existing customer IDs.
[Data Description]
The customer records are stored in the customer table in the problem9 database. The id column is a uniquei dentifier for that record.
-Output Requirements
1. Create a new table named solution stored in the problem9 database
2. Maintain the same column names and datatypes as the customer table, except store the id as a string
3. The solution table should have all of the data from the customer table, with the addition of the letter ‘A’  to the existing id values to make them unique

-Make Table
```
create external table customer9 ( id int, fname string, lname string, address string, city string, state string, zip string, birthday string) ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
LOCATION "/user/training/problem9/customer";
```

-Excute Query
```
create table solution9 as select concat("A", cast(id as string)) as id, fname, lname, address, city, state, zip, birthday from customer9;
```
-Review
```
select * from solution9 limit 10;
```

###10

-Instructions
Your boss needs specialized reports using the billing data and is constantly asking for help to write SQL queries.
Create a database view in the metastore so that your boss has customer and billing data joined.

-Data Description
The customer data exists in the customer metasore table in the problem10 database. The billing data exists in
the billing metastore table in the problem10 database. The id column is a foreign key into which customer has
the charge.

-Output Requirements
1. Create a new view named solution stored in the problem10 database
2. The new view should maintain the same column datatypes as the source tables
3. Do not return customer records that have no billing data, or billing records that have no matching customer
4. The new view should contain the following columns
5. The billdate column should only contain the date field and no time information

-Make Table
```
create external table customer10 ( id int, fname string, lname string, address string, city string, state string, zip string, birthday string) ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t' LOCATION "/user/training/problem9/customer";
create external table billing10 ( id int, charge double, tstamp string) ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t' LOCATION "/user/training/problem10/billing";
```
-Make View
```
create view solution as select c.id as id, c.fname as fname, c.lname as lname, c.city as city, c.state as state, b.charge as charge, SUBSTR(b.tstamp,0,10) as billdate from customer c join billing b on (c.id = b.id);
```
###11

-Instructions
Several analysis questions are described below and you will need to write the SQL code to answer them. You
can use whichever tool you prefer – Impala or Hive – using whichever method you like best, including shell, script, or the Hue Query Editor, to run your queries.

-Data Description
Using default database from the metastore for these queries.

-Output Requirements
1. Write the report query in the local file /home/training/problem11/solution.sql
2. Executing the solution.sql script from the hive command-line tools should generate
the report output

a. Which top three products has Dualcore sold more of than any other? Hint: Remember that if you use aGROUP BY clause, you must group by all fields listed in the SELECT clause that are not part of anaggregate function.
```
SELECT c.name, count(*) FROM orders a, order_details b, products c WHERE a.order_id =
b.order_id AND b.prod_id = c.prod_id AND c.brand = 'Dualcore' GROUP BY c.name LIMIT 3;
```
b. Calculating Revenue and Profit – write a query to show Dualcore’s revenue (total price of products sold) and profit (price minus cost) by date. Hint: The order_date column in the orders table is of type TIMESTAMP. Use the function to_date to get just the date portion of the value.
```
SELECT to_date(a.order_date), sum(c.price), sum(c.price - c.cost)
FROM orders a, order_details b, products c WHERE a.order_id = b.order_id AND b.prod_id =
c.prod_id AND c.brand = 'Dualcore' GROUP BY to_date(a.order_date);
```
c. Calculating the order Total – Which ten orders had the highest total dollar amounts?

```
SELECT a.order_id, SUM(c.price) as Total FROM orders a, order_details b, products c WHERE
a.order_id = b.order_id AND b.prod_id = c.prod_id GROUP BY a.order_id ORDER BY total desc
LIMIT 10;
```
