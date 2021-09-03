###  1. Logical Order of writing sql clauses

* SELECT (DISTINCT) , CASE WHEN, DATE FUNCTIONS, WINDOW FUNCTIONS
* FROM
* JOIN
- ON
- WHERE + WILD CARDS
- GROUP BY
- HAVING
- ORDER BY (DESC)
- LIMIT

### 2. ORDER BY

By **default**, it orders and displays numbers  and alphabets in **ascending** order, and dates in **earlier** to latest order.

Specifying `DESC` reverses ordering so, numbers and alphabets are in descending order, and dates are ordered latest first.

we can order by the column name or alias, or its index in the `SELECT` statement from left to right. (indexing starts from 1)

Index is especially helpful when ordering by an aggregate column , normally, we would have to write the whole aggregate column and not just the alias, but using column index saves us the burden.
It is also helpful when column names are just too long, or when we have to order by too many columns.


### 3.WHERE Vs. HAVING
Both WHERE and HAVING are used to filter result of a query and a query can contain both of them except that :

 1. We can't filter on aggregate columns or columns that appear in GROUP BY using the `WHERE` clause, instead, we use the `HAVING` clause and it is written between the GROUP BY  and the ORDER BY clause.
 
 2. `WHERE` is applied first on individual rows in the tables, so whichever rows meet the `WHERE` condition are grouped and put in the result set first.
 
 3. The `HAVING` clause is then applied to the rows in the result set. Only the groups that meet the `HAVING` conditions appear in the query output.


> - Wrong syntax: WHERE aggreg_function(column) > number
> - Correct syntax: **HAVING aggreg_function(column)> number**

### 4. Aggregate Functions IGNORE NULLs in columns.
This goes for all functions like COUNT, SUM, AVG,..etc
**except** `COUNT(* )`

`AVG` ignores NULL in numerator and denominator so, if we want to consider nulls as zeros, we would have to calculate `SUM(COL)/COUNT(* )` to make the average we compute ignore nulls in numerator but consider them in denominator.

### 5. GROUP BY
- ANY non-aggregate column in the `SELECT` statement must appear in the `GROUP BY` clause
- we can group by multiple columns and in the same order that they are written.
- we can write the index instead of the column name.


### 6. Filtering on dates
#### We can't filter on dates using `WHERE`, there are *date functions* to deal with that
> examples:
 - DATE_TRUNC
 - DATE_PART


 ### 7. WE can create new columns from existing ones using `CASE WHEN` in the `SELECT` statement
 ```SQL
 CASE WHEN ...... then
      WHEN ...... THEN
      ELSE....
 END AS col_alias
 ```
 `CASE WHEN` can only be used in the `SELECT` statement


### 8. SUBQUERIES
A table created from an existing table that we want to query from
```sql
SELECT
FROM (
  SUBQUERY
  )sub_alias;
```
We shouldn't write an alias for a subquery that returns an individual value , which is when a subquery is in conditional statement , it is in that case, a value to test or a set of values rather than a table.
```SQL
WHERE column_name > (subquery)
```
The only time we should alias a subquery is when it is a table.

### 9. Common Table Expressions (CTE's) or `WITH` statement
```sql
WITH table1 AS (query1),
     table2 AS (quey2),... etc
main query
```

### 10. Window FUNCTIONS
used to perform aggregate calculations on set of rows that are connected somehow but without grouping the results into a single value, it maintains the calculations in another column along with the values of  connected rows ungrouped.
**OFTEN USED FOR RUNNING TOTALS**

```SQL
SELECT function (col) OVER (PARTITION BY col ORDER BY col )
```
- OVER specifies that the function used is a window function.
- PARTITION BY specifies which set of rows that the function is going to process. it is optional so not including it will cause the function to process on all the rows.
- ORDER BY controls the order in which rows are processed by the window function including, BY DEFAULT, all rows from the start of the partition up to and including the current row  and its peers (rows having same value as current row), it doesn't have to match the order by in the main query or how the output ordering as all of the window function process happen in the background . IT is also optional. we can order multiple columns as well.

>not including the optional parts would look like empty parenthesis
```SQL
SELECT function (col) OVER ()
```

- multiple window functions can be calculated at the same time where we create an alias for the window to avoid repetition
```sql
 select wf1 over main_window,
        wf2 over main_window,
        wf3 over main_window,...

 WINDOW main_window AS (window function part)
```

#### some notes on window functions:

1. a window function, by default, processes all rows from the start of partiton up to the current row and its peers,This implies that rows that have the same values for all ORDER BY expressions will also have the same value for the result of the window function, SO this means that if we don't want to consider the peers of current row we have to specify that using onw of the frame types (ROWS) + BETWEEN UNBOUNDED PRECEDING
> Use Unbounded Preceding to make sure you don't include extra rows if 2 rows evaluate to the same thing

2. CONCEPT OF FRAMES, THEIR TYPES AND SPECIFICATIONS
we can make window functions act only on the rows of a window frame, rather than of the whole partition.
- A frame type - either ROWS, RANGE or GROUPS,
- A starting frame boundary,
- An ending frame boundary,
- An EXCLUDE clause.

> in the order by part of the window function we would define a frame.

```sql
over ( order by col + frame type + between + start_boundary + and + end_boundary )
```

#### popular window FUNCTIONS
```
lag
lead
rank
dense_rank
row_number
ntile
nth_value
cume_dist
percent_rank
first_value
```
