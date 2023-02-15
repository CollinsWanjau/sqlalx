# Databases

*  A solid db  is expected to be <b>acid</b> which means:

1. Atomicity
-> transactions are atomic, which means if a transaction fails, the
result will be like it never happened.

2. Consistency
-> You can define rules for your data, and expect that the data abides
by the rules, or else the transaction fails.

3. Isolation
-> Run two operations at the same time, and you can expect that the result
is as though they were ran one after the other.
-> As for JSON file storage; if 2 insert operations are done at the
same time, the later one will fetch an outdated collection of users
because the earlier one is not finished yet, hence overwrite the file
without the change that the earlier operation had made.

4. Durability
->Unplug your server at anytime, book it back up, and it didn't loose
any data

## Why are they called "relational" databases?

* A relation as used today is something that ties two records together,most
often across different tables.For example, say you have a blog and you have
2 tables:
-> posts, with the fields id, title and body
-> comments, with the fields id and body

* In both tables, the "id" fields are <b>primary keys</b>, because they
uniquely identify the row that they belong to.

* Once you have a relation you can do advanced things like:
-> Join tables together while querying them, which will allow you to search
for "the comments whose posts were published within the last month"

## Some more terminology around relational databases

1. Indexes

`SELECT * FROM comments WHERE post_id=12`

-> You can use an <b>Index</b> on the comments table, that applies to
the post_id column.This will "precompute" every possible SELECT query
with WHERE conditions on this column, which will update themselevs every
time you modify data, so that those calls are ready to respond quickly

-> Using an index can help the DBMS avoid scanning the entire table
for matching rows, resulting in faster query perfomance.

2. Joins

-> You can join tables together that have relations between each other,
so that you can operate on data across those tables.

-> e.g. i want the titles of all posts that have published comments

```
SELECT posts.title FROM posts JOIN comments ON posts.id = 
comments.post_id WHERE comments.published=1
```

* Perfomance is dramatically better if you manage to get the db to do
most of the work, as opposed to your application, bec. the database
knows most about your db and how to handle it most efficiently

# Basic SQL statements: DDL and DML

* DDL statements are used to build and modify the structure of your tables
and other objects in your database.When you execute a DDL statement, it
takes effect immediately.

* The <b>create table</b> statement does exactly that:
```
CREATE TABLE <table name> (
<attribute name 1> <data type 1>,
...
<attribute name n> <data type n>
```

* Data types that you will use most frequently are char strings, which might
be called VARCHAR or CHAR for variable or fixed length strings: numeric
types such as NUMBER or INTEGER, which will usually specify a precision.

* The alter table statement may be used to specify pk and fk constraints, as
well as to make modifications to the table structure

```
ALTER TABLE <table name>
ADD CONSTRAINT <constraint name> PRIMARY KEY(<attribute list>);
```

* The alter table is used to add, modify, or delete columns, constraints and
other attrs in an existing table in a relational database.

* Get used to following a specific convention of `tablename_pk`(e.g. 
Customers_pk)

* The attribute list contains the one or more attributes that form this pk;
if more than one, the names are separated by the commas.

* The fk constraint is a bit more complicated, since we have to specify both
the FK attributes in this (child) table, and the PK attributes that they
link to in the parent table.

```
ALTER TABLE <table name>
ADD CONSTRAINT <constraint name> FOREIGN KEY (<attribute list>)
REFERENCES <parent table name> (<attribute list>);
```
* Name the constraint in the form <b>childtable_parenttable_fk</b>(e.g.
Orders_Customers_fk).

* If there is more than one attribute in the FK, all of them must be included
(with commas btw) in both the FK attr list and the REFERENCES (parent table)
attr list.

* If you totally mess things up and want to start over, you can always
get rid of any object you have created with a drop statement.The syntax
is different for tables and constraints.
```
DROP TABLE <table name>;

ALTER TABLE <table name>
DROP CONSTRAINT <constraint name>
```

* This is where consistent naming comes in handy so you can just remember the
PK or FK name rather than remembering the syntax for looking up the names
in another table.

* The DROP TABLE statement gets rid of its own PK constraint, but won't work
until you separately drop any FK constraints(or child tables) that refer
to this one.

* It also gets rid of all data that was contained in the table--and it
doesn't even ask you if really want to do this

## Data Manuipulation Language

* DML statements are used to work with the data in tables.When you are
connected to most multi-user dbs you are in effect working with a private
copy of your tables that can't be seen by anyone until you are finished.

* `SELECT` is considered to be part of DML even though it just retrieves
data rather than modifying it.

* The <b>insert</b> statement:
```
INSERT INTO <table name>
VALUES (<value 1>, ...<value n>);
```

-> The comma-delimited list of values must match the table structure
exactly in the number of attributes.

-> You will need a separate INSERT statement for every row.

* The <b>update</b> is used to change the values in a table.

```
UPDATE <table name>
SET <attribute> = <expression>
WHERE <condition>;
```

* The <b>update</b> expression can be a constant, any computed value, or
even the result of a SELECT statement that returns a single row and a
single column.

* If the WHERE clasue is ommitted, then the specified attribute is set to
the same value in every row of the table.You can also set multiple attr
values at the same time with a comma-delimited list of attribute=expression

* The <b>delete</b>

```
DELETE FROM <table name>
WHERE <condition>
```

* If you are using a large multi-user system, you may need to make your DML
changes visible to the rest of the users of the db:

`COMMIT`

* If you messed up your changes in this type of sys, and you want to restore
 your private copy of the db to the way it was before you started(works
if you haven't already typed COMMIT)

`ROLLBACK`

# Basic queries: SQL and RA

## Retriving data from one table

### Retrieval with SQL

#### Basic syntax of SELECT statement

```
SELECT {attribute}+
    FROM {table}+
    [WHERE {boolean predicate to pick row}]
    [ORDER BY {attribute}+ ]
```

# SQL technique: functions

* Sometimes, the info that we need is not actually stored in the db

## Computed columns

* We can compute values from information that is in a table simply by showing
the computation in the SELECT clause.

* Each computation creates a new column in the output table

-> example, finding subtotal of a table
```
SELECT custID, orderDate, UPC, unitSalePrice * quantity
FROM orderlines;
```

-> It is awkward to read the computation as the heading, hence we can create
our own column heading or alias using the AS keyword

```
SELECT custID, orderDate, UPC, unitSalePrice * quantity AS subtotal
FROM orderlines;
```

## Aggregate functions

* SQL <b>aggregate functions</b> let us compare values based on multiple
rows in our tables 

* They are also used as part of the SELECT clause, and also creates new
columns in the output.

*  To compute this, all we need is to do is to add up all of the 
price-times-quantity computations from every line of the OrderLines. 
We will use the SUM function to do the calculation

```
SELECT SUM(unitSalePrice * quantity) AS totalsales
FROM orderlines;
```

* We'll need to add up order lines, but we need to group the total for each
order.We can do this with the <b>GROUP BY</b> clause.

* Notice that the SELECT clause and the GROUP BY clause contain exactly
the same list of attr, except for the calculation.

```
SELECT custID, orderDate, SUM(unitSalePrice * quantity) AS total
FROM orderlines
GROUP BY custID, orderDate;
```

* The COUNT function is slightly different, since it returns the number
of rows in a grouping.To count all rows, we can use the asterik to find
out how many orders were palced

```
SELECT COUNT(*)
FROM orders
```

* We can also count group of rows with identical values in a column.In this
case, COUNT will ignore NULL values in the column.Here, we'll find out how 
many times each product has been ordered.

```
SELECT prodname AS "product name"
    COUNT(prodname) AS "times ordered"
FROM products NATURAL JOIN orderlines
GROUP BY prodname;
```

* A WHERE clause can be used as usual before the GROUP BY, to eliminate rows
before the group function is executed.

* However, if we want to select output rows based on the results of the
group function, the <b>HAVING</b>.e.g asking products that have been sold
more than once:

```
SELECT prodname AS "product name"
    COUNT(prodname) AS "times ordered"
FROM products NATURAL JOIN orderlines
GROUP BY prodname
HAVING COUNT(prodname) > 1
```

# SQL technique: subqueries

* Sometimes you don't have enough info available when you design a query
to determine which rows you want, in this case you will have to find the
required info with a subquery.

-> Example: Find the name of customers who live in the same zip code 
area as Wayne Dick. We might start writing this query as we would 
any of the ones that we have already done:

```
SELECT cFirstName, cLastName
FROM customers
WHERE cZipCode = ???
```

we have to find the answer based only on the information that we have 
been given!, i.e name

```
SELECT cZipCode
FROM Customers
WHERE cFirstName = 'Wayne' AND cLastName = 'Dick'
```

* IN effect, the output of the second query becomes input to the first

* Syntantically, all we have to do is to enclose the subquery in parentheses
in the same place we will use  a constant in the WHERE clause

## What makes the big difference between a backtick and an apostrophe

```
SELECT COUNT(DISTINCT(`price`)) FROM `products`; -> Good

SELECT COUNT(DISTINCT('price')); -> Bad
```

* 'price' is a string.It never changes so the count is always 1.

* `price` refers to the column `price`.

* The inner parentheses are irrelevant.`COUNT(DISTINCT price)`.

-> `SELECT COUNT(*) FROM tbl WHERE ...`: a common way to ask how many rows.

-> `SELECT foo, COUNT(*) FROM tbl GROUP BY foo`: a common way to ask how
many rows for each distinct value of `foo`.

->`SELECT foo, COUNT(foo) FROM tbl GROUP BY foo`: is the same above except
that it does not count rows where `foo IS NULL`

-> `SELECT DISTINCT ... GROUP BY ...`: is nonsense statement.Either use
DISTINCT or use GROUP BY.


