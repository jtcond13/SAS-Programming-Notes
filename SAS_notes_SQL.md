#PROC SQL 2: the SQL#

*This is not intended as an exhaustive reference on (PROC) SQL, just the parts I need for my own reference*

###Select Statement###

The general select statement syntax is:

```
SELECT object-item <, ...object-item>
	FROM from-list
	<WHERE sql-expression>
	<GROUP BY object-item <,...object-item>>
	<HAVING sql-expression>
	<ORDER BY order-by-item <DESC>
	<,... order-by-item>>;
```

###Case Expression###
The CASE expression allows you to conditionally create columns according to the values of rows.

```
SELECT object-item <, ...object-item>
	CASE <case-operand>
		WHEN when-condition THEN result-expression
		<WHEN when-condition THEN result-expression>
		<ELSE result-expression>
	END <AS column>
FROM tale;
```

You also have the option of putting the conditional evaluation after the `CASE` case statement, which is more efficient since
the expression only have to be evaluated once.  An example of this is:

```
case year(order_date)
   when 2003 then 2003
   when 2004 then 2004
   when 2005 then 2005
   when 2006 then 2006
   else 2007
end as Sale_Year
```
###Removing Duplicate Values###
To only select unique rows in a SQL query, you can use the `DISTINCT` keyword in your select statement (e.g. `SELECT DISTINCT`).
When there are multiple columns specified in the `SELECT` statement, the `DISTINCT` keyword ensures that only unique combinations
of those columns are returned.  


###PROC SQL Options###
Adding the `Feedback` option after PROC SQL will display the expanded select statement in the SAS log.

The `DESCRIBE TABLE table-1<,... table-n>;` statement will display column attributes in the log.

The `RESET<options>;` allows you to add, drop or change the options in `PROC SQL` without restarting the procedure. The `OUTOBS=n` options
restricts the number of rows that a query outputs to a report or writes to a table.  

###Calculated and Conditional Columns###
It's possible to create columns that are the product of arithmetic operations on other columns, e.g.

```
Select Column-1, Column-1 * .2 as Column-2, ...
	from Table-1;
Quit;
```

###Subqueries ###
It is possible to nest queries within other queries, usually within a `where` or `having` clause. These queries, called Subqueries,
inner queries or sub-selects, are nested within queries by parentheses.  SAS evaluates them first before resolving the outer query.

Subqueries must return only a single column. *Correlated* subqueries require one or more values to be passed to it before it can 
resolve.  Subqueries themselves can be nested.

If a subquery returns multiple values, you must use an IN operator or comparison operator with either the ANY or ALL keyword.  The ANY
keyword specifices that at least one of a set of values obtained from a subquery must satisfy a given condition.  The ALL keyword 
specifies that all of the values obtained from a subquery must satisfy a given condition.  

###In-Line Views###
An in-line view is a query that is nested in the from clause of another query. An in-line view can return a single column or multiple columns.  Like
a subquery, it is enclosed in parentheses and does not precede a comma.

###Set Operators###
Set operators vertically combine tables.  They are:

*	Union
*	Outer Union
*	Except
*	Intersect

These operators vertically combine the intermediate result sets of queries to create a final report. 

####Rowwise####
The Union operator produces all unique rows from both result sets.  The Outer Union operator produces all rows from both
result sets.  The Except set operator produces unique rows from the first result set that are not in the second.  The
Intersect operator produces unique rows form the first result set that are in the second.  

####Columnwise####
The Union operator aligns columns by their position in both result sets.  The Outer Union operator produces all columns 
from both result sets.  The Except operator aligns columns by their position in both result sets.  The Intersect aligns columns
by their position in both result sets.  

####Other Syntax####
The `All` operator modifies the rowwise behavior of set operators by not allowing the Union, Except and Intersect operators to dedup rows.
 The `CORR` or `CORRESPONDING` operator alters the columnwise behavior by suppressing the columns with names that do not appear in both result sets. This
 operator, like `All`, only applies to the `UNION`, `EXCEPT` and `INTERSECT` operators.  SAS evaluates Intersect first, while the other set operators have the same order of precedence.
 
The syntax of the set operation is:

```
SELECT ... 
UNION | OUTER UNION | EXCEPT | INTERSECT
<ALL><CORR>
Select ...;
```

####Displaying Query Results####
You can change what a column is called in the output by adding a `'label'` after it in the `SELECT` statement.  The label
can be up to 256 characters of text.  You can also call this with `LABEL='label'`.    

To alter the format of a `PROC SQL` step, you can add the `FORMAT=formatw.d` statement in the `SELECT` statement.

You can add titles and footnotes to output using the `TITLE<n>'text'` and `FOOTNOTES<n>'text'` statements.

Note also that you can add a constant (numeric or string literal) to `PROC SQL` output using the syntax `'constant-text' <AS alias>`.

To alter report column width, use the `NOFLOW | FLOW <=n<m>>` option in which n is the minimum number of characters required
to provoke a wrapping of the column and m is the maximum. 

You can use a number of summary functions, including `SUM`, `MIN`, `MAX`, `AVG`.   You can count the number of rows using the ANSI standard `COUNT()` or `SUM()` functions
or the SAS `_N_` function. The count function counts the numbers rows in a table or subset that have a nonmissing value for the column so
long as an argument is applied.  If you supply * as an argument, the function returns the total number of rows in a table or subset. The `FREQ` and `COUNT` functions
can not accept * as an argument.

####Comparison to Data Step####
Note that the data step, passed with multiple set statements, can also be used to combine data sets vertically.  

