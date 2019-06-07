
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

The execute command itself only returns the cursor object. To see the results, you must use the fetchall method afterwards.



```python
c.fetchall()
```

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

Sadly as you can see this is slightly clunky as we do not have the column names. Doing so is a bit clunky but here it is:


```python
c.execute("""select * from employees limit 5;""")
df = pd.DataFrame(c.fetchall())
df.columns = [x[0] for x in c.description]
df.head()
```

## Additional Examples

With that, let's take a look at a few more examples:

### Selecting Customers From a Specific City


```python
c.execute("""select * from customers where city = 'Boston';""")
df = pd.DataFrame(c.fetchall())
df.columns = [x[0] for x in c.description]
df
```

### Selecting Multiple Cities


```python
c.execute("""select * from customers where city = 'Boston' or city = 'Madrid';""")
df = pd.DataFrame(c.fetchall())
df.columns = [x[0] for x in c.description]
df
```

## The Where Clause

In general, the where clause filters query results by some condition. As you are starting to see, you can also combine multiple conditions.

### Selecting Specific Columns With Complex Criteria


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

You might notice that the output of this query doesn't seem to respect our credit limit criterion. A little investigation shows that this is because the number is actually stored as a string! This is an annoying problem to encounter and also underlines the importance of setting up a database in an appropriate manner at the get go. For now, it's time to practice some of your sql querying skills!


```python
df.creditLimit.iloc[0]
```

## The Order By and Limit Clauses

Two additional keywords that you can use to refine your searches are the `ORDER BY` and `LIMIT` clauses. The order by clause allows you to sort the results by a particular feature. For example, you could sort by the `customerName` column if you wished to get results in alphabetical order. By default, `ORDER BY` is ascending. So, as with the above example, if you want the opposite, use the additional parameter `DESC`. Finally, the limit clause is typically the last argument in a SQL query and simply limits the output to a set number of results.

## Summary

In this lesson, you saw how to connect to a SQL database via python and how to subsequently execute queries against that database. Going forward, you'll continue to learn additional keywords for specifying your query parameters!
