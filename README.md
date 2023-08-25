# Filtering Data with SQL

## Introduction

After `SELECT` and `FROM`, the next SQL clause you're most likely to use as a data scientist is `WHERE`.

With just a `SELECT` expression, we can specify which **columns** we want to select, as well as transform the column values using aliases, built-in functions, and other expressions.

However if we want to filter the **rows** that we want to select, we also need to include a `WHERE` clause.

## Objectives

You will be able to:

- Retrieve a subset of records from a table using a `WHERE` clause
- Filter results using conditional operators such as `BETWEEN`, `IS NULL`, and `LIKE`
- Apply an aggregate function to the result of a filtered query

## Introduction to the `WHERE` Clause

For this section of the lesson, we'll use the Northwind database, the ERD (entity-relationship diagram) of which is shown below:  

<img src="https://curriculum-content.s3.amazonaws.com/data-science/images/Database-Schema.png">

### Northwind Data

Below, we connect to a SQLite database using the Python `sqlite3` library ([documentation here](https://docs.python.org/3/library/sqlite3.html)), then display the contents of the `employees` table:


```python
import pandas as pd
import sqlite3 
conn = sqlite3.connect('data.sqlite')
pd.read_sql("""
SELECT *
  FROM employees;
""", conn)
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
    <tr>
      <th>5</th>
      <td>1143</td>
      <td>Bow</td>
      <td>Anthony</td>
      <td>x5428</td>
      <td>abow@classicmodelcars.com</td>
      <td>1</td>
      <td>1056</td>
      <td>Sales Manager (NA)</td>
    </tr>
    <tr>
      <th>6</th>
      <td>1165</td>
      <td>Jennings</td>
      <td>Leslie</td>
      <td>x3291</td>
      <td>ljennings@classicmodelcars.com</td>
      <td>1</td>
      <td>1143</td>
      <td>Sales Rep</td>
    </tr>
    <tr>
      <th>7</th>
      <td>1166</td>
      <td>Thompson</td>
      <td>Leslie</td>
      <td>x4065</td>
      <td>lthompson@classicmodelcars.com</td>
      <td>1</td>
      <td>1143</td>
      <td>Sales Rep</td>
    </tr>
    <tr>
      <th>8</th>
      <td>1188</td>
      <td>Firrelli</td>
      <td>Julie</td>
      <td>x2173</td>
      <td>jfirrelli@classicmodelcars.com</td>
      <td>2</td>
      <td>1143</td>
      <td>Sales Rep</td>
    </tr>
    <tr>
      <th>9</th>
      <td>1216</td>
      <td>Patterson</td>
      <td>Steve</td>
      <td>x4334</td>
      <td>spatterson@classicmodelcars.com</td>
      <td>2</td>
      <td>1143</td>
      <td>Sales Rep</td>
    </tr>
    <tr>
      <th>10</th>
      <td>1286</td>
      <td>Tseng</td>
      <td>Foon Yue</td>
      <td>x2248</td>
      <td>ftseng@classicmodelcars.com</td>
      <td>3</td>
      <td>1143</td>
      <td>Sales Rep</td>
    </tr>
    <tr>
      <th>11</th>
      <td>1323</td>
      <td>Vanauf</td>
      <td>George</td>
      <td>x4102</td>
      <td>gvanauf@classicmodelcars.com</td>
      <td>3</td>
      <td>1143</td>
      <td>Sales Rep</td>
    </tr>
    <tr>
      <th>12</th>
      <td>1337</td>
      <td>Bondur</td>
      <td>Loui</td>
      <td>x6493</td>
      <td>lbondur@classicmodelcars.com</td>
      <td>4</td>
      <td>1102</td>
      <td>Sales Rep</td>
    </tr>
    <tr>
      <th>13</th>
      <td>1370</td>
      <td>Hernandez</td>
      <td>Gerard</td>
      <td>x2028</td>
      <td>ghernande@classicmodelcars.com</td>
      <td>4</td>
      <td>1102</td>
      <td>Sales Rep</td>
    </tr>
    <tr>
      <th>14</th>
      <td>1401</td>
      <td>Castillo</td>
      <td>Pamela</td>
      <td>x2759</td>
      <td>pcastillo@classicmodelcars.com</td>
      <td>4</td>
      <td>1102</td>
      <td>Sales Rep</td>
    </tr>
    <tr>
      <th>15</th>
      <td>1501</td>
      <td>Bott</td>
      <td>Larry</td>
      <td>x2311</td>
      <td>lbott@classicmodelcars.com</td>
      <td>7</td>
      <td>1102</td>
      <td>Sales Rep</td>
    </tr>
    <tr>
      <th>16</th>
      <td>1504</td>
      <td>Jones</td>
      <td>Barry</td>
      <td>x102</td>
      <td>bjones@classicmodelcars.com</td>
      <td>7</td>
      <td>1102</td>
      <td>Sales Rep</td>
    </tr>
    <tr>
      <th>17</th>
      <td>1611</td>
      <td>Fixter</td>
      <td>Andy</td>
      <td>x101</td>
      <td>afixter@classicmodelcars.com</td>
      <td>6</td>
      <td>1088</td>
      <td>Sales Rep</td>
    </tr>
    <tr>
      <th>18</th>
      <td>1612</td>
      <td>Marsh</td>
      <td>Peter</td>
      <td>x102</td>
      <td>pmarsh@classicmodelcars.com</td>
      <td>6</td>
      <td>1088</td>
      <td>Sales Rep</td>
    </tr>
    <tr>
      <th>19</th>
      <td>1619</td>
      <td>King</td>
      <td>Tom</td>
      <td>x103</td>
      <td>tking@classicmodelcars.com</td>
      <td>6</td>
      <td>1088</td>
      <td>Sales Rep</td>
    </tr>
    <tr>
      <th>20</th>
      <td>1621</td>
      <td>Nishi</td>
      <td>Mami</td>
      <td>x101</td>
      <td>mnishi@classicmodelcars.com</td>
      <td>5</td>
      <td>1056</td>
      <td>Sales Rep</td>
    </tr>
    <tr>
      <th>21</th>
      <td>1625</td>
      <td>Kato</td>
      <td>Yoshimi</td>
      <td>x102</td>
      <td>ykato@classicmodelcars.com</td>
      <td>5</td>
      <td>1621</td>
      <td>Sales Rep</td>
    </tr>
    <tr>
      <th>22</th>
      <td>1702</td>
      <td>Gerard</td>
      <td>Martin</td>
      <td>x2312</td>
      <td>mgerard@classicmodelcars.com</td>
      <td>4</td>
      <td>1102</td>
      <td>Sales Rep</td>
    </tr>
  </tbody>
</table>
</div>



When filtering data using `WHERE`, you are trying to find rows that match a specific condition. The simplest condition involves checking whether a specific column contains a specific value. In SQLite, this is done using `=`, which is similar to `==` in Python:


```python
pd.read_sql("""
SELECT *
  FROM employees
 WHERE lastName = "Patterson";
""", conn)
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
      <th>1</th>
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
      <th>2</th>
      <td>1216</td>
      <td>Patterson</td>
      <td>Steve</td>
      <td>x4334</td>
      <td>spatterson@classicmodelcars.com</td>
      <td>2</td>
      <td>1143</td>
      <td>Sales Rep</td>
    </tr>
  </tbody>
</table>
</div>



Note that we are selecting all columns (`SELECT *`) but are no longer selecting all rows. Instead, we are only selecting the 3 rows where the value of `lastName` is "Patterson".

SQL is essentially doing something like this:


```python
# Selecting all of the records in the database
result = pd.read_sql("SELECT * FROM employees;", conn)
# Create a list to store the records that match the query
employees_named_patterson = []
# Loop over all of the employees
for _, data in result.iterrows():
    # Check if the last name is "Patterson"
    if data["lastName"] == "Patterson":
        # Add to list
        employees_named_patterson.append(data)

# Display the result list as a DataFrame
pd.DataFrame(employees_named_patterson)
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
      <th>9</th>
      <td>1216</td>
      <td>Patterson</td>
      <td>Steve</td>
      <td>x4334</td>
      <td>spatterson@classicmodelcars.com</td>
      <td>2</td>
      <td>1143</td>
      <td>Sales Rep</td>
    </tr>
  </tbody>
</table>
</div>



Except SQL is designed specifically to perform these kinds of queries efficiently! **Even if you are pulling data from SQL into Python for further analysis, `SELECT * FROM <table>;` is very rarely the most efficient approach.** You should be thinking about how to get SQL to do the "heavy lifting" for you in terms of selecting, filtering, and transforming the raw data!

You can also combine `WHERE` clauses with `SELECT` statements other than `SELECT *` in order to filter rows and columns at the same time. For example:


```python
pd.read_sql("""
SELECT firstName, lastName, email
  FROM employees
 WHERE lastName = "Patterson";
""", conn)
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
      <th>firstName</th>
      <th>lastName</th>
      <th>email</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Mary</td>
      <td>Patterson</td>
      <td>mpatterso@classicmodelcars.com</td>
    </tr>
    <tr>
      <th>1</th>
      <td>William</td>
      <td>Patterson</td>
      <td>wpatterson@classicmodelcars.com</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Steve</td>
      <td>Patterson</td>
      <td>spatterson@classicmodelcars.com</td>
    </tr>
  </tbody>
</table>
</div>



`WHERE` clauses are especially powerful when combined with more-complex `SELECT` statements. Most of the time you will want to use aliases (with `AS`) in the `SELECT` statements to make the `WHERE` clauses more concise and readable.

### Selecting Employees Based on String Conditions

If we wanted to select all employees with 5 letters in their first name, that would look like this:


```python
pd.read_sql("""
SELECT *, length(firstName) AS name_length
  FROM employees
 WHERE name_length = 5;
""", conn)
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
      <th>name_length</th>
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
      <td>5</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1188</td>
      <td>Firrelli</td>
      <td>Julie</td>
      <td>x2173</td>
      <td>jfirrelli@classicmodelcars.com</td>
      <td>2</td>
      <td>1143</td>
      <td>Sales Rep</td>
      <td>5</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1216</td>
      <td>Patterson</td>
      <td>Steve</td>
      <td>x4334</td>
      <td>spatterson@classicmodelcars.com</td>
      <td>2</td>
      <td>1143</td>
      <td>Sales Rep</td>
      <td>5</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1501</td>
      <td>Bott</td>
      <td>Larry</td>
      <td>x2311</td>
      <td>lbott@classicmodelcars.com</td>
      <td>7</td>
      <td>1102</td>
      <td>Sales Rep</td>
      <td>5</td>
    </tr>
    <tr>
      <th>4</th>
      <td>1504</td>
      <td>Jones</td>
      <td>Barry</td>
      <td>x102</td>
      <td>bjones@classicmodelcars.com</td>
      <td>7</td>
      <td>1102</td>
      <td>Sales Rep</td>
      <td>5</td>
    </tr>
    <tr>
      <th>5</th>
      <td>1612</td>
      <td>Marsh</td>
      <td>Peter</td>
      <td>x102</td>
      <td>pmarsh@classicmodelcars.com</td>
      <td>6</td>
      <td>1088</td>
      <td>Sales Rep</td>
      <td>5</td>
    </tr>
  </tbody>
</table>
</div>



Or, to select all employees with the first initial of "L", that would look like this:


```python
pd.read_sql("""
SELECT *, substr(firstName, 1, 1) AS first_initial
  FROM employees
 WHERE first_initial = "L";
""", conn)
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
      <th>first_initial</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1165</td>
      <td>Jennings</td>
      <td>Leslie</td>
      <td>x3291</td>
      <td>ljennings@classicmodelcars.com</td>
      <td>1</td>
      <td>1143</td>
      <td>Sales Rep</td>
      <td>L</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1166</td>
      <td>Thompson</td>
      <td>Leslie</td>
      <td>x4065</td>
      <td>lthompson@classicmodelcars.com</td>
      <td>1</td>
      <td>1143</td>
      <td>Sales Rep</td>
      <td>L</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1337</td>
      <td>Bondur</td>
      <td>Loui</td>
      <td>x6493</td>
      <td>lbondur@classicmodelcars.com</td>
      <td>4</td>
      <td>1102</td>
      <td>Sales Rep</td>
      <td>L</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1501</td>
      <td>Bott</td>
      <td>Larry</td>
      <td>x2311</td>
      <td>lbott@classicmodelcars.com</td>
      <td>7</td>
      <td>1102</td>
      <td>Sales Rep</td>
      <td>L</td>
    </tr>
  </tbody>
</table>
</div>



**Important note:** Just like in Python, you can compare numbers in SQL just by typing the number (e.g. `name_length = 5`) but if you want to compare to a string value, you need to surround the value with quotes (e.g. `first_initial = "L"`). If you forget the quotes, you will get an error, because SQL will interpret it as a variable name rather than a hard-coded value:


```python
pd.read_sql("""
SELECT *, substr(firstName, 1, 1) AS first_initial
  FROM employees
 WHERE first_initial = L;
""", conn)
```


    ---------------------------------------------------------------------------

    OperationalError                          Traceback (most recent call last)

    ~/opt/anaconda3/envs/learn-env/lib/python3.8/site-packages/pandas/io/sql.py in execute(self, *args, **kwargs)
       2017         try:
    -> 2018             cur.execute(*args, **kwargs)
       2019             return cur


    OperationalError: no such column: L

    
    The above exception was the direct cause of the following exception:


    DatabaseError                             Traceback (most recent call last)

    <ipython-input-7-083650d31057> in <module>
    ----> 1 pd.read_sql("""
          2 SELECT *, substr(firstName, 1, 1) AS first_initial
          3   FROM employees
          4  WHERE first_initial = L;
          5 """, conn)


    ~/opt/anaconda3/envs/learn-env/lib/python3.8/site-packages/pandas/io/sql.py in read_sql(sql, con, index_col, coerce_float, params, parse_dates, columns, chunksize)
        562 
        563     if isinstance(pandas_sql, SQLiteDatabase):
    --> 564         return pandas_sql.read_query(
        565             sql,
        566             index_col=index_col,


    ~/opt/anaconda3/envs/learn-env/lib/python3.8/site-packages/pandas/io/sql.py in read_query(self, sql, index_col, coerce_float, params, parse_dates, chunksize, dtype)
       2076 
       2077         args = _convert_params(sql, params)
    -> 2078         cursor = self.execute(*args)
       2079         columns = [col_desc[0] for col_desc in cursor.description]
       2080 


    ~/opt/anaconda3/envs/learn-env/lib/python3.8/site-packages/pandas/io/sql.py in execute(self, *args, **kwargs)
       2028 
       2029             ex = DatabaseError(f"Execution failed on sql '{args[0]}': {exc}")
    -> 2030             raise ex from exc
       2031 
       2032     @staticmethod


    DatabaseError: Execution failed on sql '
    SELECT *, substr(firstName, 1, 1) AS first_initial
      FROM employees
     WHERE first_initial = L;
    ': no such column: L


### Selecting Order Details Based on Price

Below we select all order details where the price each, rounded to the nearest integer, is 30 dollars:


```python
pd.read_sql("""
SELECT *, CAST(round(priceEach) AS INTEGER) AS rounded_price_int
  FROM orderDetails
 WHERE rounded_price_int = 30;
""", conn)
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
      <th>orderNumber</th>
      <th>productCode</th>
      <th>quantityOrdered</th>
      <th>priceEach</th>
      <th>orderLineNumber</th>
      <th>rounded_price_int</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>10104</td>
      <td>S24_2840</td>
      <td>44</td>
      <td>30.41</td>
      <td>10</td>
      <td>30</td>
    </tr>
    <tr>
      <th>1</th>
      <td>10173</td>
      <td>S24_1937</td>
      <td>31</td>
      <td>29.87</td>
      <td>9</td>
      <td>30</td>
    </tr>
    <tr>
      <th>2</th>
      <td>10184</td>
      <td>S24_2840</td>
      <td>42</td>
      <td>30.06</td>
      <td>7</td>
      <td>30</td>
    </tr>
    <tr>
      <th>3</th>
      <td>10280</td>
      <td>S24_1937</td>
      <td>20</td>
      <td>29.87</td>
      <td>12</td>
      <td>30</td>
    </tr>
    <tr>
      <th>4</th>
      <td>10332</td>
      <td>S24_1937</td>
      <td>45</td>
      <td>29.87</td>
      <td>6</td>
      <td>30</td>
    </tr>
    <tr>
      <th>5</th>
      <td>10367</td>
      <td>S24_1937</td>
      <td>23</td>
      <td>29.54</td>
      <td>13</td>
      <td>30</td>
    </tr>
    <tr>
      <th>6</th>
      <td>10380</td>
      <td>S24_1937</td>
      <td>32</td>
      <td>29.87</td>
      <td>4</td>
      <td>30</td>
    </tr>
  </tbody>
</table>
</div>



### Selecting Orders Based on Date

We can use the `strftime` function to select all orders placed in January of any year:


```python
pd.read_sql("""
SELECT *, strftime("%m", orderDate) AS month
  FROM orders
 WHERE month = "01";
""", conn)
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
      <th>orderNumber</th>
      <th>orderDate</th>
      <th>requiredDate</th>
      <th>shippedDate</th>
      <th>status</th>
      <th>comments</th>
      <th>customerNumber</th>
      <th>month</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>10100</td>
      <td>2003-01-06</td>
      <td>2003-01-13</td>
      <td>2003-01-10</td>
      <td>Shipped</td>
      <td></td>
      <td>363</td>
      <td>01</td>
    </tr>
    <tr>
      <th>1</th>
      <td>10101</td>
      <td>2003-01-09</td>
      <td>2003-01-18</td>
      <td>2003-01-11</td>
      <td>Shipped</td>
      <td>Check on availability.</td>
      <td>128</td>
      <td>01</td>
    </tr>
    <tr>
      <th>2</th>
      <td>10102</td>
      <td>2003-01-10</td>
      <td>2003-01-18</td>
      <td>2003-01-14</td>
      <td>Shipped</td>
      <td></td>
      <td>181</td>
      <td>01</td>
    </tr>
    <tr>
      <th>3</th>
      <td>10103</td>
      <td>2003-01-29</td>
      <td>2003-02-07</td>
      <td>2003-02-02</td>
      <td>Shipped</td>
      <td></td>
      <td>121</td>
      <td>01</td>
    </tr>
    <tr>
      <th>4</th>
      <td>10104</td>
      <td>2003-01-31</td>
      <td>2003-02-09</td>
      <td>2003-02-01</td>
      <td>Shipped</td>
      <td></td>
      <td>141</td>
      <td>01</td>
    </tr>
    <tr>
      <th>5</th>
      <td>10208</td>
      <td>2004-01-02</td>
      <td>2004-01-11</td>
      <td>2004-01-04</td>
      <td>Shipped</td>
      <td></td>
      <td>146</td>
      <td>01</td>
    </tr>
    <tr>
      <th>6</th>
      <td>10209</td>
      <td>2004-01-09</td>
      <td>2004-01-15</td>
      <td>2004-01-12</td>
      <td>Shipped</td>
      <td></td>
      <td>347</td>
      <td>01</td>
    </tr>
    <tr>
      <th>7</th>
      <td>10210</td>
      <td>2004-01-12</td>
      <td>2004-01-22</td>
      <td>2004-01-20</td>
      <td>Shipped</td>
      <td></td>
      <td>177</td>
      <td>01</td>
    </tr>
    <tr>
      <th>8</th>
      <td>10211</td>
      <td>2004-01-15</td>
      <td>2004-01-25</td>
      <td>2004-01-18</td>
      <td>Shipped</td>
      <td></td>
      <td>406</td>
      <td>01</td>
    </tr>
    <tr>
      <th>9</th>
      <td>10212</td>
      <td>2004-01-16</td>
      <td>2004-01-24</td>
      <td>2004-01-18</td>
      <td>Shipped</td>
      <td></td>
      <td>141</td>
      <td>01</td>
    </tr>
    <tr>
      <th>10</th>
      <td>10213</td>
      <td>2004-01-22</td>
      <td>2004-01-28</td>
      <td>2004-01-27</td>
      <td>Shipped</td>
      <td>Difficult to negotiate with customer. We need ...</td>
      <td>489</td>
      <td>01</td>
    </tr>
    <tr>
      <th>11</th>
      <td>10214</td>
      <td>2004-01-26</td>
      <td>2004-02-04</td>
      <td>2004-01-29</td>
      <td>Shipped</td>
      <td></td>
      <td>458</td>
      <td>01</td>
    </tr>
    <tr>
      <th>12</th>
      <td>10215</td>
      <td>2004-01-29</td>
      <td>2004-02-08</td>
      <td>2004-02-01</td>
      <td>Shipped</td>
      <td>Customer requested that FedEx Ground is used f...</td>
      <td>475</td>
      <td>01</td>
    </tr>
    <tr>
      <th>13</th>
      <td>10362</td>
      <td>2005-01-05</td>
      <td>2005-01-16</td>
      <td>2005-01-10</td>
      <td>Shipped</td>
      <td></td>
      <td>161</td>
      <td>01</td>
    </tr>
    <tr>
      <th>14</th>
      <td>10363</td>
      <td>2005-01-06</td>
      <td>2005-01-12</td>
      <td>2005-01-10</td>
      <td>Shipped</td>
      <td></td>
      <td>334</td>
      <td>01</td>
    </tr>
    <tr>
      <th>15</th>
      <td>10364</td>
      <td>2005-01-06</td>
      <td>2005-01-17</td>
      <td>2005-01-09</td>
      <td>Shipped</td>
      <td></td>
      <td>350</td>
      <td>01</td>
    </tr>
    <tr>
      <th>16</th>
      <td>10365</td>
      <td>2005-01-07</td>
      <td>2005-01-18</td>
      <td>2005-01-11</td>
      <td>Shipped</td>
      <td></td>
      <td>320</td>
      <td>01</td>
    </tr>
    <tr>
      <th>17</th>
      <td>10366</td>
      <td>2005-01-10</td>
      <td>2005-01-19</td>
      <td>2005-01-12</td>
      <td>Shipped</td>
      <td></td>
      <td>381</td>
      <td>01</td>
    </tr>
    <tr>
      <th>18</th>
      <td>10367</td>
      <td>2005-01-12</td>
      <td>2005-01-21</td>
      <td>2005-01-16</td>
      <td>Resolved</td>
      <td>This order was disputed and resolved on 2/1/20...</td>
      <td>205</td>
      <td>01</td>
    </tr>
    <tr>
      <th>19</th>
      <td>10368</td>
      <td>2005-01-19</td>
      <td>2005-01-27</td>
      <td>2005-01-24</td>
      <td>Shipped</td>
      <td>Can we renegotiate this one?</td>
      <td>124</td>
      <td>01</td>
    </tr>
    <tr>
      <th>20</th>
      <td>10369</td>
      <td>2005-01-20</td>
      <td>2005-01-28</td>
      <td>2005-01-24</td>
      <td>Shipped</td>
      <td></td>
      <td>379</td>
      <td>01</td>
    </tr>
    <tr>
      <th>21</th>
      <td>10370</td>
      <td>2005-01-20</td>
      <td>2005-02-01</td>
      <td>2005-01-25</td>
      <td>Shipped</td>
      <td></td>
      <td>276</td>
      <td>01</td>
    </tr>
    <tr>
      <th>22</th>
      <td>10371</td>
      <td>2005-01-23</td>
      <td>2005-02-03</td>
      <td>2005-01-25</td>
      <td>Shipped</td>
      <td></td>
      <td>124</td>
      <td>01</td>
    </tr>
    <tr>
      <th>23</th>
      <td>10372</td>
      <td>2005-01-26</td>
      <td>2005-02-05</td>
      <td>2005-01-28</td>
      <td>Shipped</td>
      <td></td>
      <td>398</td>
      <td>01</td>
    </tr>
    <tr>
      <th>24</th>
      <td>10373</td>
      <td>2005-01-31</td>
      <td>2005-02-08</td>
      <td>2005-02-06</td>
      <td>Shipped</td>
      <td></td>
      <td>311</td>
      <td>01</td>
    </tr>
  </tbody>
</table>
</div>



We can also check to see if any orders were shipped late (`shippedDate` after `requiredDate`, i.e. the number of days late is a positive number):


```python
pd.read_sql("""
SELECT *, julianday(shippedDate) - julianday(requiredDate) AS days_late
  FROM orders
 WHERE days_late > 0;
""", conn)
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
      <th>orderNumber</th>
      <th>orderDate</th>
      <th>requiredDate</th>
      <th>shippedDate</th>
      <th>status</th>
      <th>comments</th>
      <th>customerNumber</th>
      <th>days_late</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>10165</td>
      <td>2003-10-22</td>
      <td>2003-10-31</td>
      <td>2003-12-26</td>
      <td>Shipped</td>
      <td>This order was on hold because customers's cre...</td>
      <td>148</td>
      <td>56.0</td>
    </tr>
  </tbody>
</table>
</div>



That was the last query in this lesson using the Northwind data, so let's close that connection:


```python
conn.close()
```

## Conditional Operators in SQL

In all of the above queries, we used the `=` operator to check if we had an exact match for a given value. However, what if you wanted to select the order details where the price was at least 30 dollars? Or all of the orders that don't currently have a shipped date?

We'll need some more advanced conditional operators for that.

Some important ones to know are:

* `!=` ("not equal to")
  * Similar to `not` combined with `==` in Python
* `>` ("greater than")
  * Similar to `>` in Python
* `>=` ("greater than or equal to")
  * Similar to `>=` in Python
* `<` ("less than")
  * Similar to `<` in Python
* `<=` ("less than or equal to")
  * Similar to `<=` in Python
* `AND`
  * Similar to `and` in Python
* `OR`
  * Similar to `or` in Python
* `BETWEEN`
  * Similar to placing a value between two values with `<=` and `and` in Python, e.g. `(2 <= x) and (x <= 5)`
* `IN`
  * Similar to `in` in Python
* `LIKE`
  * Uses wildcards to find similar strings. No direct equivalent in Python, but similar to some Bash terminal commands.

### Cats Data

For this section as the queries get more advanced we'll be using a simpler database called `pets_database.db` containing a table called `cats`.

The `cats` table is populated with the following data:


|id  |name     |age    |breed              |owner_id   |
|----|---------|-------|-------------------|-----------|
|1   |Maru     |3.0    |Scottish Fold      |1.0        |
|2   |Hana     |1.0    |Tabby              |1.0        |
|3   |Lil' Bub |5.0    |American Shorthair |NaN        |
|4   |Moe      |10.0   |Tabby              |NaN        |
|5   |Patches  |2.0    |Calico             |NaN        |
|6   |None     |NaN    |Tabby              |NaN        |

Below we make a new database connection and read all of the data from this table:


```python
conn = sqlite3.connect('pets_database.db')
pd.read_sql("SELECT * FROM cats;", conn)
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
      <th>id</th>
      <th>name</th>
      <th>age</th>
      <th>breed</th>
      <th>owner_id</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>Maru</td>
      <td>3.0</td>
      <td>Scottish Fold</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>Hana</td>
      <td>1.0</td>
      <td>Tabby</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>Lil' Bub</td>
      <td>5.0</td>
      <td>American Shorthair</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4</td>
      <td>Moe</td>
      <td>10.0</td>
      <td>Tabby</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4</th>
      <td>5</td>
      <td>Patches</td>
      <td>2.0</td>
      <td>Calico</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>5</th>
      <td>6</td>
      <td>None</td>
      <td>NaN</td>
      <td>Tabby</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
</div>



### `WHERE` Code-Along

In this exercise, you'll walk through executing a handful of common and handy SQL queries that use `WHERE` with conditional operators. We'll start by giving you an example of what this type of query looks like, then have you type a query specifically related to the `cats` table.

#### `WHERE` with `>=`

For the `=`, `!=`, `<`, `<=`, `>`, and `>=` operators, the query looks like:

```sql
SELECT column(s)
  FROM table_name
 WHERE column_name operator value;
```

> Note: The example above is not valid SQL, it is a template for how the queries are constructed

Type this SQL query between the quotes below to select all cats who are at least 5 years old:

```sql
SELECT *
  FROM cats
 WHERE age >= 5;
```


```python
pd.read_sql("""

""", conn)
```

This should return:

|id  |name     |age    |breed              |owner_id   |
|----|---------|-------|-------------------|-----------|
|3   |Lil' Bub |5.0    |American Shorthair |None       |
|4   |Moe      |10.0   |Tabby              |None       |

#### `WHERE` with `BETWEEN`

If you wanted to select all rows with values in a range, you _could_ do this by combining the `<=` and `AND` operators. However, since this is such a common task in SQL, there is a shorter and more efficient command specifically for this purpose, called `BETWEEN`.

A typical query with `BETWEEN` looks like:

```sql
SELECT column_name(s)
  FROM table_name
 WHERE column_name BETWEEN value1 AND value2;
```

> Note that `BETWEEN` is an **inclusive** range, so the returned values can match the boundary values (not like `range()` in Python)

Let's say you need to select the names of all of the cats whose age is between 1 and 3. Type this SQL query between the quotes below to select all cats who are in this age range:

```sql
SELECT *
  FROM cats
 WHERE age BETWEEN 1 AND 3;
```


```python
pd.read_sql("""

""", conn)
```

This should return:

|id  |name     |age    |breed              |owner_id   |
|----|---------|-------|-------------------|-----------|
|1   |Maru     |3.0    |Scottish Fold      |1.0        |
|2   |Hana     |1.0    |Tabby              |1.0        |
|5   |Patches  |2.0    |Calico             |NaN        |

#### `WHERE` Column Is Not `NULL`

`NULL` in SQL represents missing data. It is similar to `None` in Python or `NaN` in NumPy or pandas. However, we use the `IS` operator to check if something is `NULL`, not the `=` operator (or `IS NOT` instead of `!=`).

To check if a value is `NULL` (or not), the query looks like:

```sql
SELECT column(s)
  FROM table_name
 WHERE column_name IS (NOT) NULL;
```

> You might have noticed when we selected all rows of `cats`, some owner IDs were `NaN`, then in the above query they are `None` instead. This is a subtle difference where Python/pandas is converting SQL `NULL` values to `NaN` when there are numbers in other rows, and converting to `None` when all of the returned values are `NULL`. This is a subtle difference that you don't need to memorize; it is just highlighted to demonstrate that the operators we use in SQL are _similar_ to Python operators, but not quite the same.

If we want to select all cats that don't currently belong to an owner, we want to select all cats where the `owner_id` is `NULL`.

Type this SQL query between the quotes below to select all cats that don't currently belong to an owner:

```sql
SELECT *
  FROM cats
 WHERE owner_id IS NULL;
```


```python
pd.read_sql("""

""", conn)
```

This should return:

|id  |name     |age    |breed              |owner_id   |
|----|---------|-------|-------------------|-----------|
|3   |Lil' Bub |5.0    |American Shorthair |None       |
|4   |Moe      |10.0   |Tabby              |None       |
|5   |Patches  |2.0    |Calico             |None       |
|6   |None     |NaN    |Tabby              |None       |

#### `WHERE` with `LIKE`

The `LIKE` operator is very helpful for writing SQL queries with messy data. It uses _wildcards_ to specify which parts of the string query need to be an exact match and which parts can be variable.

When using `LIKE`, a query looks like:

```sql
SELECT column(s)
  FROM table_name
 WHERE column_name LIKE 'string_with_wildcards';
```

The most common wildcard you'll see is `%`. This is similar to the `*` wildcard in Bash or regex: it means zero or more characters with any value can be in that position.

So for example, if we want all cats with names that start with "M", we could use a query containing `M%`. This means that we're looking for matches that start with one character "M" (or "m", since this is a case-insensitive query in SQLite) and then zero or more characters that can have any value.

Type this SQL query between the quotes below to select all cats with names that start with "M" (or "m"):

```sql
SELECT *
  FROM cats
 WHERE name LIKE 'M%';
```


```python
pd.read_sql("""

""", conn)
```

This should return:

|id  |name     |age    |breed              |owner_id   |
|----|---------|-------|-------------------|-----------|
|1   |Maru     |3.0    |Scottish Fold      |1.0        |
|4   |Moe      |10.0   |Tabby              |NaN        |

Note that we also could have used the `substr` SQL built-in function here to perform the same task:

```sql
SELECT *
  FROM cats
 WHERE substr(name, 1, 1) = "M";
```

Unlike in Python where:

> There should be one-- and preferably only one --obvious way to do it. *(Zen of Python)*

there will often be multiple valid approaches to writing the same SQL query. Sometimes one will be more efficient than the other, and sometimes the only difference will be a matter of preference.

The other wildcard used for comparing strings is `_`, which means exactly one character, with any value.

For example, if we wanted to select all cats with four-letter names where the second letter was "a", we could use `_a__`.

Type this SQL query between the quotes below to select all cats with names where the second letter is "a" and the name is four letters long:

```sql
SELECT *
  FROM cats
 WHERE name LIKE '_a__';
```


```python
pd.read_sql("""

""", conn)
```

This should return:

|id  |name     |age    |breed              |owner_id   |
|----|---------|-------|-------------------|-----------|
|1   |Maru     |3      |Scottish Fold      |1          |
|2   |Hana     |1      |Tabby              |1          |

Again, we could have done this using `length` and `substr`, although it would be much less concise:

```sql
SELECT *
  FROM cats
 WHERE length(name) = 4 AND substr(name, 2, 1) = "a";
```

These examples are a bit silly, but you can imagine how this technique would help to write queries between multiple datasets where the names don't quite match exactly! You can combine `%` and `_` in your string to narrow and expand your query results as needed.

#### `SELECT` with `COUNT`

Now, let's talk about the SQL aggregate function `COUNT`.

**SQL aggregate functions** are SQL statements that can get the average of a column's values, retrieve the minimum and maximum values from a column, sum values in a column, or count a number of records that meet certain conditions. You can learn more about these SQL aggregators [here](http://www.sqlclauses.com/sql+aggregate+functions) and [here](http://zetcode.com/db/sqlite/select/).

For now, we'll just focus on `COUNT`, which counts the number of records that meet a certain condition. Here's a standard SQL query using `COUNT`:

```sql
SELECT COUNT(column_name)
  FROM table_name
 WHERE conditional_statement;
```

Let's try it out and count the number of cats who have an `owner_id` of `1`. Type this SQL query between the quotes below:

```sql
SELECT COUNT(owner_id)
  FROM cats
 WHERE owner_id = 1;
```


```python
pd.read_sql("""

""", conn)
```

This should return:

|  |COUNT(owner_id) |
|--|----------------|
|0 |2               |

## Note on `SELECT`

We are now familiar with this syntax:

```sql
SELECT name
  FROM cats;
```

However, you may not know that this can be written like this as well:

```sql
SELECT cats.name
  FROM cats;
```

Both return the same data.

SQLite allows us to explicitly state the `tableName.columnName` you want to select. This is particularly useful when you want data from two different tables.

Imagine you have another table called `dogs` with a column containing all of the dog names:

```sql
CREATE TABLE dogs (
	id   INTEGER PRIMARY KEY,
	name TEXT
);
```

```sql
INSERT INTO dogs (name)
VALUES ("Clifford");
```


If you want to get the names of all the dogs and cats, you can no longer run a query with just the column name.
`SELECT name FROM cats,dogs;` will return `Error: ambiguous column name: name`.

Instead, you must explicitly follow the `tableName.columnName` syntax.
```sql
SELECT cats.name, dogs.name
  FROM cats, dogs;
```

You may see this in the future. Don't let it trip you up!



```python
conn.close()
```

## Summary

In this lesson, you saw how to filter the resulting rows of a SQL query using a `WHERE` clause that checked whether a given column was equal to a specific value. You also got a basic introduction to aggregate functions by seeing an example of `COUNT`, and dove deeper into some conditional operators including `BETWEEN` and `LIKE`.
