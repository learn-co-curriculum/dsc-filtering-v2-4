# Selecting Data

## Introduction

Now that you've gotten a brief introduction to SQL, its time to get some hands-on practice connecting to a database via Python and executing some queries.

## Objectives

You will be able to:
- Connect to a SQL database using Python
- Retrieve all information from a SQL table
- Retrieve a subset of records from a table using a `WHERE` clause
- Write SQL queries to filter and order results
- Retrieve a subset of columns from a table

## Connecting to a Database Using Python

SQLite databases are stored as files on disk. The one we will be using in this lesson is called `data.sqlite`.


```python
! ls
```

    CONTRIBUTING.md README.md       [34mimages[m[m
    LICENSE.md      data.sqlite     index.ipynb


(Here the file extension is `.sqlite` but you will also see examples ending with `.db`)

If we try to read from this file without using any additional libraries, we will get a bunch of garbled nonsense, since this file is encoded as bytes and not plain text:


```python
with open("data.sqlite", "rb") as f:
    print(f.read(100))
```

    b'SQLite format 3\x00\x10\x00\x01\x01\x00@  \x00\x00\x00\x10\x00\x00\x008\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x08\x00\x00\x00\x04\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x01\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x10\x00.\x05B'


### Connection

Instead, we will use the `sqlite3` module ([documentation here](https://docs.python.org/3/library/sqlite3.html)). The way that this module works is that we start by opening a *connection* to the database with `sqlite3.connect`:


```python
import sqlite3 
conn = sqlite3.connect('data.sqlite')
```

We will use this connection throughout the lesson, then close it at the end.

Let's look at some more attributes of the connection:


```python
print("Data type:", type(conn))
print("Uncommitted changes:", conn.in_transaction)
print("Total changes:", conn.total_changes)
```

    Data type: <class 'sqlite3.Connection'>
    Uncommitted changes: False
    Total changes: 0


As you can see, since we have only opened the connection and not performed any queries, there are no uncommitted changes and 0 total changes so far. Let's continue on and create a *cursor*.

### Cursor

A cursor object is what can actually execute SQL commands. You create it by calling `.cursor()` on the connection.


```python
cur = conn.cursor()
```


```python
type(cur)
```




    sqlite3.Cursor



### Exploring the Schema

Let's use the cursor to find out what tables are contained in this database. This requires two steps:

1. Executing the query (`.execute()`)
2. Fetching the results (`.fetchone()`, `.fetchmany()`, or `.fetchall()`)

This is because some SQL commands (e.g. deleting data) do not require results to be fetched, just commands to be executed. So the interface only fetches the results if you ask for them.


```python
# Execute the query
cur.execute("""SELECT name FROM sqlite_master WHERE type = 'table';""")
# Fetch the result and store it in table_names
table_names = cur.fetchall()
table_names
```




    [('orderdetails',),
     ('payments',),
     ('offices',),
     ('customers',),
     ('orders',),
     ('productlines',),
     ('products',),
     ('employees',)]



So now we know the names of the tables. What if we want to know the schema of the `employees` table?


```python
cur.execute("""SELECT sql FROM sqlite_master WHERE type = 'table' AND name = 'employees';""")
employee_columns = cur.fetchone()
employee_columns
```




    ('CREATE TABLE `employees` (`employeeNumber`, `lastName`, `firstName`, `extension`, `email`, `officeCode`, `reportsTo`, `jobTitle`)',)



Ok, now we know the names of the columns!

## ERD Overview

The database that you've just connected to is the same database you have seen previously, containing data about orders, employeers, etc. Here's an overview of the database:  

<img src="images/Database-Schema.png">

If we want to get all information about the first 5 employee records, we might do something like this (`*` means all columns):


```python
cur.execute("""SELECT * FROM employees LIMIT 5;""")
cur.fetchall()
```




    [('1002',
      'Murphy',
      'Diane',
      'x5800',
      'dmurphy@classicmodelcars.com',
      '1',
      '',
      'President'),
     ('1056',
      'Patterson',
      'Mary',
      'x4611',
      'mpatterso@classicmodelcars.com',
      '1',
      '1002',
      'VP Sales'),
     ('1076',
      'Firrelli',
      'Jeff',
      'x9273',
      'jfirrelli@classicmodelcars.com',
      '1',
      '1002',
      'VP Marketing'),
     ('1088',
      'Patterson',
      'William',
      'x4871',
      'wpatterson@classicmodelcars.com',
      '6',
      '1056',
      'Sales Manager (APAC)'),
     ('1102',
      'Bondur',
      'Gerard',
      'x5408',
      'gbondur@classicmodelcars.com',
      '4',
      '1056',
      'Sale Manager (EMEA)')]



Because `.execute()` returns the cursor object, it also possible to combine the previous two lines into one line, like so:


```python
cur.execute("""SELECT * FROM employees LIMIT 5;""").fetchall()
```




    [('1002',
      'Murphy',
      'Diane',
      'x5800',
      'dmurphy@classicmodelcars.com',
      '1',
      '',
      'President'),
     ('1056',
      'Patterson',
      'Mary',
      'x4611',
      'mpatterso@classicmodelcars.com',
      '1',
      '1002',
      'VP Sales'),
     ('1076',
      'Firrelli',
      'Jeff',
      'x9273',
      'jfirrelli@classicmodelcars.com',
      '1',
      '1002',
      'VP Marketing'),
     ('1088',
      'Patterson',
      'William',
      'x4871',
      'wpatterson@classicmodelcars.com',
      '6',
      '1056',
      'Sales Manager (APAC)'),
     ('1102',
      'Bondur',
      'Gerard',
      'x5408',
      'gbondur@classicmodelcars.com',
      '4',
      '1056',
      'Sale Manager (EMEA)')]



### Quick Note on String Syntax

When working with strings, you may have previously seen a `'string'`, a `"string"`, a `'''string'''`, or a `"""string"""`. While all of these are strings, the triple quotes have the added functionality of being able to use multiple lines within the same string as well as to use single quotes within the string. Sometimes, SQL queries can be much longer than others, in which case it's helpful to use new lines for readability. Here's the same example, this time with the string spread out onto multiple lines:


```python
first_five_employees_query = """
SELECT *
FROM employees
LIMIT 5
;
"""

cur.execute(first_five_employees_query).fetchall()
```




    [('1002',
      'Murphy',
      'Diane',
      'x5800',
      'dmurphy@classicmodelcars.com',
      '1',
      '',
      'President'),
     ('1056',
      'Patterson',
      'Mary',
      'x4611',
      'mpatterso@classicmodelcars.com',
      '1',
      '1002',
      'VP Sales'),
     ('1076',
      'Firrelli',
      'Jeff',
      'x9273',
      'jfirrelli@classicmodelcars.com',
      '1',
      '1002',
      'VP Marketing'),
     ('1088',
      'Patterson',
      'William',
      'x4871',
      'wpatterson@classicmodelcars.com',
      '6',
      '1056',
      'Sales Manager (APAC)'),
     ('1102',
      'Bondur',
      'Gerard',
      'x5408',
      'gbondur@classicmodelcars.com',
      '4',
      '1056',
      'Sale Manager (EMEA)')]



## Wrapping Results into Pandas DataFrames

Often, a more convenient output will be to turn these results into pandas DataFrames. One way to do this would be to wrap the `c.fetchall()` output with a pandas DataFrame constructor:


```python
import pandas as pd
```


```python
df = pd.DataFrame(cur.execute(first_five_employees_query).fetchall())
df
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>0</th>
      <th>1</th>
      <th>2</th>
      <th>3</th>
      <th>4</th>
      <th>5</th>
      <th>6</th>
      <th>7</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1002</td>
      <td>Murphy</td>
      <td>Diane</td>
      <td>x5800</td>
      <td>dmurphy@classicmodelcars.com</td>
      <td>1</td>
      <td></td>
      <td>President</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1056</td>
      <td>Patterson</td>
      <td>Mary</td>
      <td>x4611</td>
      <td>mpatterso@classicmodelcars.com</td>
      <td>1</td>
      <td>1002</td>
      <td>VP Sales</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1076</td>
      <td>Firrelli</td>
      <td>Jeff</td>
      <td>x9273</td>
      <td>jfirrelli@classicmodelcars.com</td>
      <td>1</td>
      <td>1002</td>
      <td>VP Marketing</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1088</td>
      <td>Patterson</td>
      <td>William</td>
      <td>x4871</td>
      <td>wpatterson@classicmodelcars.com</td>
      <td>6</td>
      <td>1056</td>
      <td>Sales Manager (APAC)</td>
    </tr>
    <tr>
      <th>4</th>
      <td>1102</td>
      <td>Bondur</td>
      <td>Gerard</td>
      <td>x5408</td>
      <td>gbondur@classicmodelcars.com</td>
      <td>4</td>
      <td>1056</td>
      <td>Sale Manager (EMEA)</td>
    </tr>
  </tbody>
</table>
</div>



Sadly as you can see this is slightly clunky as we do not have the column names. Pandas just automatically assigns the numbers 0 through 7. 

We can access the column names by calling `cur.description`, like so:


```python
cur.description
```




    (('employeeNumber', None, None, None, None, None, None),
     ('lastName', None, None, None, None, None, None),
     ('firstName', None, None, None, None, None, None),
     ('extension', None, None, None, None, None, None),
     ('email', None, None, None, None, None, None),
     ('officeCode', None, None, None, None, None, None),
     ('reportsTo', None, None, None, None, None, None),
     ('jobTitle', None, None, None, None, None, None))



Then using a list comprehension, assign the column names after we instantiate the dataframe:


```python
df.columns = [x[0] for x in cur.description]
df
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>employeeNumber</th>
      <th>lastName</th>
      <th>firstName</th>
      <th>extension</th>
      <th>email</th>
      <th>officeCode</th>
      <th>reportsTo</th>
      <th>jobTitle</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1002</td>
      <td>Murphy</td>
      <td>Diane</td>
      <td>x5800</td>
      <td>dmurphy@classicmodelcars.com</td>
      <td>1</td>
      <td></td>
      <td>President</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1056</td>
      <td>Patterson</td>
      <td>Mary</td>
      <td>x4611</td>
      <td>mpatterso@classicmodelcars.com</td>
      <td>1</td>
      <td>1002</td>
      <td>VP Sales</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1076</td>
      <td>Firrelli</td>
      <td>Jeff</td>
      <td>x9273</td>
      <td>jfirrelli@classicmodelcars.com</td>
      <td>1</td>
      <td>1002</td>
      <td>VP Marketing</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1088</td>
      <td>Patterson</td>
      <td>William</td>
      <td>x4871</td>
      <td>wpatterson@classicmodelcars.com</td>
      <td>6</td>
      <td>1056</td>
      <td>Sales Manager (APAC)</td>
    </tr>
    <tr>
      <th>4</th>
      <td>1102</td>
      <td>Bondur</td>
      <td>Gerard</td>
      <td>x5408</td>
      <td>gbondur@classicmodelcars.com</td>
      <td>4</td>
      <td>1056</td>
      <td>Sale Manager (EMEA)</td>
    </tr>
  </tbody>
</table>
</div>



Even better, there is a pandas method directly designed for reading from SQL databases ([documentation here](https://pandas.pydata.org/docs/reference/api/pandas.read_sql.html)). Instead of using the cursor, all you need is the connection variable:


```python
pd.read_sql(first_five_employees_query, conn)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>employeeNumber</th>
      <th>lastName</th>
      <th>firstName</th>
      <th>extension</th>
      <th>email</th>
      <th>officeCode</th>
      <th>reportsTo</th>
      <th>jobTitle</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1002</td>
      <td>Murphy</td>
      <td>Diane</td>
      <td>x5800</td>
      <td>dmurphy@classicmodelcars.com</td>
      <td>1</td>
      <td></td>
      <td>President</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1056</td>
      <td>Patterson</td>
      <td>Mary</td>
      <td>x4611</td>
      <td>mpatterso@classicmodelcars.com</td>
      <td>1</td>
      <td>1002</td>
      <td>VP Sales</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1076</td>
      <td>Firrelli</td>
      <td>Jeff</td>
      <td>x9273</td>
      <td>jfirrelli@classicmodelcars.com</td>
      <td>1</td>
      <td>1002</td>
      <td>VP Marketing</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1088</td>
      <td>Patterson</td>
      <td>William</td>
      <td>x4871</td>
      <td>wpatterson@classicmodelcars.com</td>
      <td>6</td>
      <td>1056</td>
      <td>Sales Manager (APAC)</td>
    </tr>
    <tr>
      <th>4</th>
      <td>1102</td>
      <td>Bondur</td>
      <td>Gerard</td>
      <td>x5408</td>
      <td>gbondur@classicmodelcars.com</td>
      <td>4</td>
      <td>1056</td>
      <td>Sale Manager (EMEA)</td>
    </tr>
  </tbody>
</table>
</div>



It is still useful to be aware of the cursor construct in case you ever need to develop Python code that fetches one result at a time, or is a command other than `SELECT`. But in general if you know that the end result is creating a pandas dataframe to display the result, you don't really need to interface with the cursor directly.

Note that we can also use `SELECT` to select only certain columns, and those will be reflected in the dataframe column names:


```python
pd.read_sql("""SELECT lastname, firstName FROM employees;""", conn)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>lastName</th>
      <th>firstName</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Murphy</td>
      <td>Diane</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Patterson</td>
      <td>Mary</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Firrelli</td>
      <td>Jeff</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Patterson</td>
      <td>William</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Bondur</td>
      <td>Gerard</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Bow</td>
      <td>Anthony</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Jennings</td>
      <td>Leslie</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Thompson</td>
      <td>Leslie</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Firrelli</td>
      <td>Julie</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Patterson</td>
      <td>Steve</td>
    </tr>
    <tr>
      <th>10</th>
      <td>Tseng</td>
      <td>Foon Yue</td>
    </tr>
    <tr>
      <th>11</th>
      <td>Vanauf</td>
      <td>George</td>
    </tr>
    <tr>
      <th>12</th>
      <td>Bondur</td>
      <td>Loui</td>
    </tr>
    <tr>
      <th>13</th>
      <td>Hernandez</td>
      <td>Gerard</td>
    </tr>
    <tr>
      <th>14</th>
      <td>Castillo</td>
      <td>Pamela</td>
    </tr>
    <tr>
      <th>15</th>
      <td>Bott</td>
      <td>Larry</td>
    </tr>
    <tr>
      <th>16</th>
      <td>Jones</td>
      <td>Barry</td>
    </tr>
    <tr>
      <th>17</th>
      <td>Fixter</td>
      <td>Andy</td>
    </tr>
    <tr>
      <th>18</th>
      <td>Marsh</td>
      <td>Peter</td>
    </tr>
    <tr>
      <th>19</th>
      <td>King</td>
      <td>Tom</td>
    </tr>
    <tr>
      <th>20</th>
      <td>Nishi</td>
      <td>Mami</td>
    </tr>
    <tr>
      <th>21</th>
      <td>Kato</td>
      <td>Yoshimi</td>
    </tr>
    <tr>
      <th>22</th>
      <td>Gerard</td>
      <td>Martin</td>
    </tr>
  </tbody>
</table>
</div>



## The `WHERE` Clause

Now that we have the general syntax down, let's try for some more complex queries!

In general, the `WHERE` clause filters `SELECT` query results by some condition.

### Selecting Customers from a Specific City

Note that because the query is surrounded by triple quotes (`"""`) we can use single quotes (`'`) around the string literals within the query, e.g. `'Boston'`. You need to put quotes around strings in SQL just like you do in Python, so that it is interpreted as a string and not a variable name.


```python
pd.read_sql("""SELECT * FROM customers WHERE city = 'Boston';""", conn)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>customerNumber</th>
      <th>customerName</th>
      <th>contactLastName</th>
      <th>contactFirstName</th>
      <th>phone</th>
      <th>addressLine1</th>
      <th>addressLine2</th>
      <th>city</th>
      <th>state</th>
      <th>postalCode</th>
      <th>country</th>
      <th>salesRepEmployeeNumber</th>
      <th>creditLimit</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>362</td>
      <td>Gifts4AllAges.com</td>
      <td>Yoshido</td>
      <td>Juri</td>
      <td>6175559555</td>
      <td>8616 Spinnaker Dr.</td>
      <td></td>
      <td>Boston</td>
      <td>MA</td>
      <td>51003</td>
      <td>USA</td>
      <td>1216</td>
      <td>41900.00</td>
    </tr>
    <tr>
      <th>1</th>
      <td>495</td>
      <td>Diecast Collectables</td>
      <td>Franco</td>
      <td>Valarie</td>
      <td>6175552555</td>
      <td>6251 Ingle Ln.</td>
      <td></td>
      <td>Boston</td>
      <td>MA</td>
      <td>51003</td>
      <td>USA</td>
      <td>1188</td>
      <td>85100.00</td>
    </tr>
  </tbody>
</table>
</div>



### Selecting Multiple Cities

As you are starting to see, you can also combine multiple conditions.


```python
pd.read_sql("""SELECT * FROM customers WHERE city = 'Boston' OR city = 'Madrid';""", conn)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>customerNumber</th>
      <th>customerName</th>
      <th>contactLastName</th>
      <th>contactFirstName</th>
      <th>phone</th>
      <th>addressLine1</th>
      <th>addressLine2</th>
      <th>city</th>
      <th>state</th>
      <th>postalCode</th>
      <th>country</th>
      <th>salesRepEmployeeNumber</th>
      <th>creditLimit</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>141</td>
      <td>Euro+ Shopping Channel</td>
      <td>Freyre</td>
      <td>Diego</td>
      <td>(91) 555 94 44</td>
      <td>C/ Moralzarzal, 86</td>
      <td></td>
      <td>Madrid</td>
      <td></td>
      <td>28034</td>
      <td>Spain</td>
      <td>1370</td>
      <td>227600.00</td>
    </tr>
    <tr>
      <th>1</th>
      <td>237</td>
      <td>ANG Resellers</td>
      <td>Camino</td>
      <td>Alejandra</td>
      <td>(91) 745 6555</td>
      <td>Gran Vía, 1</td>
      <td></td>
      <td>Madrid</td>
      <td></td>
      <td>28001</td>
      <td>Spain</td>
      <td></td>
      <td>0.00</td>
    </tr>
    <tr>
      <th>2</th>
      <td>344</td>
      <td>CAF Imports</td>
      <td>Fernandez</td>
      <td>Jesus</td>
      <td>+34 913 728 555</td>
      <td>Merchants House</td>
      <td>27-30 Merchant's Quay</td>
      <td>Madrid</td>
      <td></td>
      <td>28023</td>
      <td>Spain</td>
      <td>1702</td>
      <td>59600.00</td>
    </tr>
    <tr>
      <th>3</th>
      <td>362</td>
      <td>Gifts4AllAges.com</td>
      <td>Yoshido</td>
      <td>Juri</td>
      <td>6175559555</td>
      <td>8616 Spinnaker Dr.</td>
      <td></td>
      <td>Boston</td>
      <td>MA</td>
      <td>51003</td>
      <td>USA</td>
      <td>1216</td>
      <td>41900.00</td>
    </tr>
    <tr>
      <th>4</th>
      <td>458</td>
      <td>Corrida Auto Replicas, Ltd</td>
      <td>Sommer</td>
      <td>Martín</td>
      <td>(91) 555 22 82</td>
      <td>C/ Araquil, 67</td>
      <td></td>
      <td>Madrid</td>
      <td></td>
      <td>28023</td>
      <td>Spain</td>
      <td>1702</td>
      <td>104600.00</td>
    </tr>
    <tr>
      <th>5</th>
      <td>465</td>
      <td>Anton Designs, Ltd.</td>
      <td>Anton</td>
      <td>Carmen</td>
      <td>+34 913 728555</td>
      <td>c/ Gobelas, 19-1 Urb. La Florida</td>
      <td></td>
      <td>Madrid</td>
      <td></td>
      <td>28023</td>
      <td>Spain</td>
      <td></td>
      <td>0.00</td>
    </tr>
    <tr>
      <th>6</th>
      <td>495</td>
      <td>Diecast Collectables</td>
      <td>Franco</td>
      <td>Valarie</td>
      <td>6175552555</td>
      <td>6251 Ingle Ln.</td>
      <td></td>
      <td>Boston</td>
      <td>MA</td>
      <td>51003</td>
      <td>USA</td>
      <td>1188</td>
      <td>85100.00</td>
    </tr>
  </tbody>
</table>
</div>



## The `ORDER BY` and `LIMIT` Clauses

Two additional keywords that you can use to refine your searches are the `ORDER BY` and `LIMIT` clauses.

The `ORDER BY` clause allows you to sort the results by a particular feature. For example, you could sort by the `customerName` column if you wished to get results in alphabetical order.

By default, `ORDER BY` is ascending. So, to continue the previous example, if you want the customers in reverse alphabetical order, use the additional parameter `DESC` immediately after whatever you are ordering by.

Finally, the limit clause is typically the last argument in a SQL query and simply limits the output to a set number of results, as seen with the employee data above. This is especially useful when you are performing initial data exploration and do not need to see thousands or millions of results.

### Selecting Specific Columns with Complex Criteria

This query demonstrates essentially all of the SQL features we have covered so far. It is asking for the number, name, city, and credit limit for all customers located in Boston or Madrid whose credit limit is above 50,000.00. Then it sorts by the credit limit and limits to the top 15 results.


```python
complex_query = """
SELECT customerNumber, customerName, city, creditLimit
FROM customers
WHERE (city = 'Boston' OR city = 'Madrid') AND (creditLimit >= 50000.00)
ORDER BY creditLimit DESC
LIMIT 15
;"""
df = pd.read_sql(complex_query, conn)
df
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>customerNumber</th>
      <th>customerName</th>
      <th>city</th>
      <th>creditLimit</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>495</td>
      <td>Diecast Collectables</td>
      <td>Boston</td>
      <td>85100.00</td>
    </tr>
    <tr>
      <th>1</th>
      <td>344</td>
      <td>CAF Imports</td>
      <td>Madrid</td>
      <td>59600.00</td>
    </tr>
    <tr>
      <th>2</th>
      <td>362</td>
      <td>Gifts4AllAges.com</td>
      <td>Boston</td>
      <td>41900.00</td>
    </tr>
    <tr>
      <th>3</th>
      <td>141</td>
      <td>Euro+ Shopping Channel</td>
      <td>Madrid</td>
      <td>227600.00</td>
    </tr>
    <tr>
      <th>4</th>
      <td>458</td>
      <td>Corrida Auto Replicas, Ltd</td>
      <td>Madrid</td>
      <td>104600.00</td>
    </tr>
    <tr>
      <th>5</th>
      <td>237</td>
      <td>ANG Resellers</td>
      <td>Madrid</td>
      <td>0.00</td>
    </tr>
    <tr>
      <th>6</th>
      <td>465</td>
      <td>Anton Designs, Ltd.</td>
      <td>Madrid</td>
      <td>0.00</td>
    </tr>
  </tbody>
</table>
</div>



You might notice that the output of this query doesn't seem to respect our credit limit criterion. There are results here where the credit limit is *not* over 50,000.00.

A little investigation shows that this is because the number is actually stored as a string! 


```python
df["creditLimit"].iloc[0]
```




    '85100.00'




```python
print(df["creditLimit"].dtype)
```

    object


Let's do some additional investigation to figure out what happened.

One additional technique we can use to understand the schema of a SQLITE table is the `PRAGMA` `table_info` command. You can read more about it in the [SQLite docs](https://www.sqlite.org/pragma.html#pragma_table_info). Essentially it shows you the full schema of a given table:


```python
pd.read_sql("""PRAGMA table_info(customers)""", conn, index_col="cid")
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>name</th>
      <th>type</th>
      <th>notnull</th>
      <th>dflt_value</th>
      <th>pk</th>
    </tr>
    <tr>
      <th>cid</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>customerNumber</td>
      <td></td>
      <td>0</td>
      <td>None</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>customerName</td>
      <td></td>
      <td>0</td>
      <td>None</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>contactLastName</td>
      <td></td>
      <td>0</td>
      <td>None</td>
      <td>0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>contactFirstName</td>
      <td></td>
      <td>0</td>
      <td>None</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>phone</td>
      <td></td>
      <td>0</td>
      <td>None</td>
      <td>0</td>
    </tr>
    <tr>
      <th>5</th>
      <td>addressLine1</td>
      <td></td>
      <td>0</td>
      <td>None</td>
      <td>0</td>
    </tr>
    <tr>
      <th>6</th>
      <td>addressLine2</td>
      <td></td>
      <td>0</td>
      <td>None</td>
      <td>0</td>
    </tr>
    <tr>
      <th>7</th>
      <td>city</td>
      <td></td>
      <td>0</td>
      <td>None</td>
      <td>0</td>
    </tr>
    <tr>
      <th>8</th>
      <td>state</td>
      <td></td>
      <td>0</td>
      <td>None</td>
      <td>0</td>
    </tr>
    <tr>
      <th>9</th>
      <td>postalCode</td>
      <td></td>
      <td>0</td>
      <td>None</td>
      <td>0</td>
    </tr>
    <tr>
      <th>10</th>
      <td>country</td>
      <td></td>
      <td>0</td>
      <td>None</td>
      <td>0</td>
    </tr>
    <tr>
      <th>11</th>
      <td>salesRepEmployeeNumber</td>
      <td></td>
      <td>0</td>
      <td>None</td>
      <td>0</td>
    </tr>
    <tr>
      <th>12</th>
      <td>creditLimit</td>
      <td></td>
      <td>0</td>
      <td>None</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
</div>



According to this, none of the columns actually have a data type specified (the `type` column is empty) and none of the columns is marked as the primary key (`pk` column). SQLite is defaulting to treating them like strings — even `creditLimit`, which we clearly want to treat as a number — because the schema doesn't specify their types.

This is an annoying problem to encounter and also underlines the importance of setting up a database in an appropriate manner at the get-go. Sometimes you will encounter an issue like this and you won't be able to do all of the desired filtering in SQL, and instead will need to use pandas or some other technique for your final analysis.

### Bonus: Database Administration

In this case, you have full control over the database since it's just a file on disk, on a computer you control. You can do some database administration and make a correctly-typed copy of `creditLimit` called `creditLimitNumeric`, so that the above complex query works.

***Important note:*** it is okay if you don't understand this part. Much of the time, data scientists are only given read access (so they can write `SELECT` queries) and are not responsible for database administration. This is just to give an example of what it takes to fix a database that is not set up correctly.

First, note that because all of our queries so far have been `SELECT` queries, we still have not made any changes. It's a good idea to keep track of these attributes of `conn` as you attempt to perform any database administration.


```python
print("Uncommitted changes:", conn.in_transaction)
print("Total changes:", conn.total_changes)
```

    Uncommitted changes: False
    Total changes: 0


Now we can write a query that will alter the database structure (adding a new column `creditLimitNumeric`):


```python
add_column = """
ALTER TABLE customers
ADD COLUMN creditLimitNumeric REAL;
"""
cur.execute(add_column)
```




    <sqlite3.Cursor at 0x10838fea0>



Then copy all of the `creditLimit` values to the new `creditLimitNumeric` column:


```python
fill_values = """
UPDATE customers
SET creditLimitNumeric = creditLimit
;
"""
cur.execute(fill_values)
```




    <sqlite3.Cursor at 0x10838fea0>



Now if we check the attributes of `conn`, we do have some uncommitted changes:


```python
print("Uncommitted changes:", conn.in_transaction)
print("Total changes:", conn.total_changes)
```

    Uncommitted changes: True
    Total changes: 122


So we need to commit them:


```python
conn.commit()
```


```python
print("Uncommitted changes:", conn.in_transaction)
print("Total changes:", conn.total_changes)
```

    Uncommitted changes: False
    Total changes: 122


Now we can look at our table info again:


```python
pd.read_sql("""PRAGMA table_info(customers)""", conn, index_col="cid")
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>name</th>
      <th>type</th>
      <th>notnull</th>
      <th>dflt_value</th>
      <th>pk</th>
    </tr>
    <tr>
      <th>cid</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>customerNumber</td>
      <td></td>
      <td>0</td>
      <td>None</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>customerName</td>
      <td></td>
      <td>0</td>
      <td>None</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>contactLastName</td>
      <td></td>
      <td>0</td>
      <td>None</td>
      <td>0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>contactFirstName</td>
      <td></td>
      <td>0</td>
      <td>None</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>phone</td>
      <td></td>
      <td>0</td>
      <td>None</td>
      <td>0</td>
    </tr>
    <tr>
      <th>5</th>
      <td>addressLine1</td>
      <td></td>
      <td>0</td>
      <td>None</td>
      <td>0</td>
    </tr>
    <tr>
      <th>6</th>
      <td>addressLine2</td>
      <td></td>
      <td>0</td>
      <td>None</td>
      <td>0</td>
    </tr>
    <tr>
      <th>7</th>
      <td>city</td>
      <td></td>
      <td>0</td>
      <td>None</td>
      <td>0</td>
    </tr>
    <tr>
      <th>8</th>
      <td>state</td>
      <td></td>
      <td>0</td>
      <td>None</td>
      <td>0</td>
    </tr>
    <tr>
      <th>9</th>
      <td>postalCode</td>
      <td></td>
      <td>0</td>
      <td>None</td>
      <td>0</td>
    </tr>
    <tr>
      <th>10</th>
      <td>country</td>
      <td></td>
      <td>0</td>
      <td>None</td>
      <td>0</td>
    </tr>
    <tr>
      <th>11</th>
      <td>salesRepEmployeeNumber</td>
      <td></td>
      <td>0</td>
      <td>None</td>
      <td>0</td>
    </tr>
    <tr>
      <th>12</th>
      <td>creditLimit</td>
      <td></td>
      <td>0</td>
      <td>None</td>
      <td>0</td>
    </tr>
    <tr>
      <th>13</th>
      <td>creditLimitNumeric</td>
      <td>REAL</td>
      <td>0</td>
      <td>None</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
</div>



Ok, all the way at the bottom we see there is a column `creditLimitNumeric` with `type` of `REAL` (the SQLite name for floating point values). Let's try our complex query again:


```python
# query edited to refer to creditLimitNumeric
complex_query = """
SELECT customerNumber, customerName, city, creditLimitNumeric
FROM customers
WHERE (city = 'Boston' OR city = 'Madrid') AND (creditLimitNumeric >= 50000.00)
ORDER BY creditLimitNumeric DESC
LIMIT 15
;"""
df = pd.read_sql(complex_query, conn)
df
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>customerNumber</th>
      <th>customerName</th>
      <th>city</th>
      <th>creditLimitNumeric</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>141</td>
      <td>Euro+ Shopping Channel</td>
      <td>Madrid</td>
      <td>227600.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>458</td>
      <td>Corrida Auto Replicas, Ltd</td>
      <td>Madrid</td>
      <td>104600.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>495</td>
      <td>Diecast Collectables</td>
      <td>Boston</td>
      <td>85100.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>344</td>
      <td>CAF Imports</td>
      <td>Madrid</td>
      <td>59600.0</td>
    </tr>
  </tbody>
</table>
</div>




```python
print(df['creditLimitNumeric'].dtype)
```

    float64


It worked!

Note that this was a fairly conservative, cautious approach to editing the database. We could have dumped the entire contents into a temp database, then read them back in with the appropriate schema, if we wanted to keep the name `creditLimit` while also setting the appropriate data type. But that kind of operation carries more risk compared to making a copy like this. Most of the time as a data scientist (not a database administrator), these are the kinds of changes you want to make: not true administrative overhauls, but just enough modification so that your query will work how you need it to.

Now we can go ahead and close our database connection. Similar to working with CSV or JSON files, it is mainly important to close the connection if you are writing data to the file/database, but it's a best practice to close it regardless.


```python
conn.close()
```

## Summary

In this lesson, you saw how to connect to a SQL database via Python and how to subsequently execute queries against that database. Going forward, you'll continue to learn additional keywords for specifying your query parameters!
