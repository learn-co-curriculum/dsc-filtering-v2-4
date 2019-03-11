
# Selecting Data

## Introduction

Now that you've gotten a brief introduction to SQL, its time to get some hands on pratice connecting to a database via Python and executing to some queries.

## Objectives

You will be able to:

* Understand the basic structure of a `SELECT` statement in SQL

## Connecting To a Database

First let's connect to our database by importing sqlite3 and running the following commands in our notebook.
```python 
import sqlite3
conn = sqlite3.connect('data.sqlite')
c = conn.cursor()
```


```python
# connect database and create cursor here
import sqlite3 
conn = sqlite3.connect('data.sqlite')
c = conn.cursor()
```

## Schema Overview

The database that you're now connected to is the same as from the previous introduction. Here's an overview of the database:  

<img src="images/Database-Schema.png">


## Querying Via the Connection

Now that you're connected to the database, let's take a look at how you can query the data within.

With your cursor object, you can execute queries


```python
c.execute("""select * from employees limit 5;""")
```




    <sqlite3.Cursor at 0x1efebff5ea0>



The execute command itself only returns the cursor object. To see the results, you must use the fetchall method afterwards.



```python
c.fetchall()
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



## Wrapping Results Into Pandas DataFrames

Often, a more convenient output will be to turn these results into pandas DataFrames. To do this, you simply wrap the `c.fetchall()` output with a pandas DataFrame constructor:


```python
import pandas as pd
```


```python
c.execute("""select * from employees limit 5;""")
df = pd.DataFrame(c.fetchall())
df.head()
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



Sadly as you can see this is slightly clunky as we do not have the column names. Doing so is a bit clunky but here it is:


```python
c.execute("""select * from employees limit 5;""")
df = pd.DataFrame(c.fetchall())
df.columns = [x[0] for x in c.description]
df.head()
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



## Additional Examples

With that, let's take a look at a few more examples:

### Selecting Customers From a Specific City


```python
c.execute("""select * from customers where city = 'Boston';""")
df = pd.DataFrame(c.fetchall())
df.columns = [x[0] for x in c.description]
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


```python
c.execute("""select * from customers where city = 'Boston' or city = 'Madrid';""")
df = pd.DataFrame(c.fetchall())
df.columns = [x[0] for x in c.description]
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



## The Where Clause

In general, the where clause filters query results by some condition. As you are starting to see, you can also combine multiple conditions.

### Selecting Specific Columns With a Complex Criteria


```python
c.execute("""select customerNumber, customerName, city, creditLimit
             from customers
             where (city = 'Boston' or city = 'Madrid')
                   and (creditLimit >= 50000.00)
             order by creditLimit desc
             limit 15
             ;""")
df = pd.DataFrame(c.fetchall())
df.columns = [x[0] for x in c.description]
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



You might notice that the output of this query doesn't seem to respect our credit limit criterion. A little investigation shows that this is because the number is actually stored as a string! This is an annoying problem to encounter and also underlines the importance of setting up a database in an appropriate manner at the get go. For now, it's time to practice some of your sql querying skills!


```python
df.creditLimitditLimit.iloc[0]
```




    '85100.00'



## The Order By and Limit Clauses

Two additional keywords that you can use to refine your searches are the `ORDER BY` and `LIMIT` clauses. The order by clause allows you to sort the results by a particular feature. For example, you could sort by the `customerName` column if you wished to get results in alphabetical order. By default, `ORDER BY` is ascending. So, as with the above example, if you want the opposite, use the additional parameter `DESC`. Finally, the limit clause is typically the last argument in a SQL query and simply limits the output to a set number of results.

## Summary

In this lesson, you saw how to connect to a SQL database via python and how to subsequently execute queries against that database. Going forward, you'll continue to learn additional keywords for specifying your query parameters!
