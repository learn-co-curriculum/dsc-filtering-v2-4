# Inserting, Selecting, Updating, and Deleting Database Rows

## Overview

In this lesson, we'll cover different ways to manipulate and select data from SQL database tables.

## Objectives

1. Use the `INSERT INTO` command to insert data (i.e. rows) into a database table
2. Use `SELECT` statements to select data from a database table
3. Use the `WHERE` Clause to select data from specific table rows
4. Use comparison operators, like `<` or `>`, to select specific data
3. Use `UPDATE` statements to update data within a database table
4. Use `DELETE` statements to delete data from a database table

## Setting Up Our Database

In this code along, we'll be creating a `cats` table in a `pets_database.db`. So, let's navigate to our terminal and get started.

First let's create our `pets_database` by running the following command.
```sql
sqlite3 pets_database.db
```

Now that we have a database, let's create our `cats` table along with `id`, `name`, `age` and `breed` columns.

```sql
CREATE TABLE cats (
	id INTEGER PRIMARY KEY,
	name TEXT,
	age INTEGER,
	breed TEXT
);
```

Okay, let's start storing some cats.

### Code Along I: INSERT INTO

In your terminal, in the sqlite prompt, type the following:

```sql
INSERT INTO cats (name, age, breed) VALUES ('Maru', 3, 'Scottish Fold');
```

We use the `INSERT INTO` command, followed by the name of the table to which we want to add data. Then, in parentheses, we put the column names that we will be filling with data. This is followed by the `VALUES` keyword, which is accompanied by a parentheses enclosed list of the values that correspond to each column name.

**Important:** Note that we *didn't specify* the "id" column name or value. Since we created the `cats` table with an "id" column whose type is `INTEGER PRIMARY KEY`, we don't have to specify the id column values when we insert data. Primary Key columns are auto-incrementing. As long as you have defined an id column with a data type of `INTEGER PRIMARY KEY`, a newly inserted row's id column will be automatically given the correct value.

Let's add a few more cats to our table. This time we'll do this via our text editor. Create a file, `01_insert_cats_into_cats_table.sql`. Use two `INSERT INTO` statements to insert the following cats into the table:

|name|age|breed|
|----|---|-----|
|"Lil\' Bub"|5|"American Shorthair"|
|"Hannah"|1|"Tabby"|

Each `INSERT INTO` statement gets its own line in the `.sql` file in your text editor. Each line needs to end with a `;`. Run the file with the following code in your terminal:

```bash
sqlite3 pets_database.db < 01_insert_cats_into_cats_table.sql
```
**NOTE:** This is a bash command, run this from your bash not sqlite console.

Now, we'll learn how to `SELECT` data from a table, which will help us to confirm that we inserted the above data correctly.

## Selecting Data

Now that we've inserted some data into our `cats` table, we likely want to read that data. This is where the `SELECT` statement comes in. We use it to retrieve database data, or rows.

### Code Along II: SELECT FROM

A basic `SELECT` statement works like this:

```sql
SELECT [names of columns we are going to select] FROM [table we are selecting from];
```

We specify the names of the columns we want to SELECT and then tell SQL the table we want to select them FROM.

We want to select all the rows in our table, and we want to return the data stored in any and all columns in those rows. To do this, we could pass the name of each column explicitly:

```sql
SELECT id, name, age, breed FROM cats;
```

Which should give us back:

```bash
1|Maru|3|Scottish Fold
2|Lil\' Bub|5|American Shorthair
3|Hannah|1|Tabby
```

A faster way to get data from every column in our table is to use a special selector, known commonly as the 'wildcard', `*` selector. The `*` selector means: "Give me all the data from all the columns for all of the cats" Using the wildcard, we can `SELECT` all the data from all of the columns in the cats table like this:

```sql
SELECT * FROM cats;
```

Now let's try out some more specific `SELECT` statements:

#### Selecting by Column Names

To select just certain columns from a table, use the following:

```sql
SELECT name FROM cats;
```
That should return the following:

```bash
Maru
Lil\' Bub
Hannah
```

You can even select more than one column name at a time. For example, try out:

```sql
SELECT name, age FROM cats;
```


**Top-Tip:** If you have duplicate data (for example, two cats with the same name) and you only want to select unique values, you can use the `DISTINCT` keyword. For example:

```sql
SELECT DISTINCT name FROM cats;
```


#### Selecting Based on Conditions: The `WHERE` Clause
What happens when we want to retrieve a specific table row? For example the row that belongs to Maru? Or to retrieve all the baby cats who are younger than two years old? We can use the `WHERE` keyword to select data based on specific conditions. Here's an example of a boilerplate `SELECT` statement using a `WHERE` clause.

```sql
SELECT * FROM [table name] WHERE [column name] = [some value];
```

Let's retrieve *just Maru* from our `cats` table:

```sql
sqlite> SELECT * FROM cats WHERE name = "Maru";
```
That statement should return the following:

```bash
1|Maru|3|Scottish Fold
```

We can also use comparison operators, like `<` or `>` to select specific data. Let's give it a shot. Use the following statement to select the young cats:

```sql
SELECT * FROM cats WHERE age < 2;
```

**Advanced:** The SQL statements we're learning here will eventually be used to integrate the applications you'll build with a database. For example, it's easy to imagine a web application that has many users. When a user signs into your app, you'll need to access your database and select the user that matches the credentials an individual is using to log in.

## Updating Data

Let's talk about updating, or changing, data in our table rows. We do this with the `UPDATE` keyword.

### Code Along III: UPDATE

A boilerplate `UPDATE` statement looks like this:

```sql
UPDATE [table name] SET [column name] = [new value] WHERE [column name] = [value];
```

The `UPDATE` statement uses a `WHERE` clause to grab the row you want to update. It identifies the table name you are looking in and resets the data in a particular column to a new value.

Let's update one of our cats. Turns out Maru's friend Hannah is actually Maru's friend *Hana*. Let's update that row to change the name to the correct spelling:

```sql
sqlite> UPDATE cats SET name = "Hana" WHERE name = "Hannah";
```

One last thing before we move on: deleting table rows.

## Deleting Data

To delete table rows, we use the `DELETE` keyword.

### Code Along IV: DELETE

A boilerplate `DELETE` statement looks like this:

```sql
DELETE FROM [table name] WHERE [column name] = [value];
```

Let's go ahead and delete Lil' Bub from our `cats` table (sorry Lil' Bub):

```sql
sqlite> DELETE FROM cats WHERE id = 2;
```

Notice that this time we selected the row to delete using the Primary Key column. Remember that every table row has a Primary Key column that is unique. Lil' Bub was the second row in the database and thus had an id of `2`.

<p data-visibility='hidden'>View <a href='https://learn.co/lessons/sql-insert-select-update-code-along' title='Inserting, Selecting, Updating, and Deleting Database Rows'>Inserting, Selecting, Updating, and Deleting Database Rows</a> on Learn.co and start learning to code for free.</p>
