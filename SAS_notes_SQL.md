# PROC SQL 2: the SQL #

*This is not intended as an exhaustive reference on (PROC) SQL*

### Select Statement ###

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

### Case Expression ###
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

### Removing Duplicate Values ###
To only select unique rows in a SQL query, you can use the `DISTINCT` keyword in your select statement (e.g. `SELECT DISTINCT`).
When there are multiple columns specified in the `SELECT` statement, the `DISTINCT` keyword ensures that only unique combinations
of those columns are returned.  


### PROC SQL Options ###
You add options to `PROC SQL` in the first line of the step (i.e. `PROC SQL <option(s)>;`).  Note that certain options only apply to certain
ODS destinations (HTML, PDF, etc.). 

The `OUTOBS=n` option restricts the number of observations written to the table.  If this cuts off the results of your query, a Warning
will be written to the log. The `INOBS=n` option sets a limit of n rows from each source table that contributes to a query.

The `NUMBER | NONUMBER` options control whether or not a column displaying the row number is created.  This formats the ODS output but 
does not affect the underlying table.

The `NODOUBLE | DOUBLE` options control whether listing ODS output is single or double spaced (defaults to `NODOUBLE`).

To alter report column width, use the `NOFLOW | FLOW <=n<m>>` option in which n is the minimum number of characters required
to provoke a wrapping of the column and m is the maximum. Defaults to `NOFLOW`.

Adding the `Feedback` option after PROC SQL will display the expanded select statement in the SAS log.

The `DESCRIBE TABLE table-1<,... table-n>;` statement will display column attributes in the log.

The `RESET<options>;` allows you to add, drop or change the options in `PROC SQL` without restarting the procedure. The `OUTOBS=n` options
restricts the number of rows that a query outputs to a report or writes to a table.  

The `NOEXEC` option prevents the step from executing, but just checks the code for errors.  

`PRINT | NOPRINT` controls whether the results of a `SELECT` statement are displayed as a report.  `NOSTIMER|STIMER` controls whether
`PROC SQL` writes resource utilizations statistics to the SAS log.  `NOFLOW|FLOW` controls text wrapping in character columns.

`NOSYMBOLGEN|SYMBOLGEN` writes macro variable values to the SAS log.  `FEEDBACK` works similarly, but rewrites the entire `PROC SQL` statement
with variable values substituted for macro variables.

### Calculated and Conditional Columns ###
It's possible to create columns that are the product of arithmetic operations on other columns, e.g.

```
Select Column-1, Column-1 * .2 as Column-2, ...
	from Table-1;
Quit;
```

### Subqueries ###
It is possible to nest queries within other queries, usually within a `where` or `having` clause. These queries, called Subqueries,
inner queries or sub-selects, are nested within queries by parentheses.  SAS evaluates them first before resolving the outer query.

Subqueries must return only a single column. **Correlated** subqueries require one or more values to be passed to it before it can 
resolve.  Subqueries themselves can be nested.

If a subquery returns multiple values, you must use an IN operator or comparison operator with either the ANY or ALL keyword.  The ANY
keyword specifices that at least one of a set of values obtained from a subquery must satisfy a given condition.  The ALL keyword 
specifies that all of the values obtained from a subquery must satisfy a given condition.  

### In-Line Views ###
An in-line view is a query that is nested in the from clause of another query. An in-line view can return a single column or multiple columns.  Like
a subquery, it is enclosed in parentheses and does not precede a comma.

### Set Operators ###
Set operators vertically combine tables.  They are:

*	Union
*	Outer Union
*	Except
*	Intersect

These operators vertically combine the intermediate result sets of queries to create a final report. 

#### Rowwise ####
The Union operator produces all unique rows from both result sets.  The Outer Union operator produces all rows from both
result sets.  The Except set operator produces unique rows from the first result set that are not in the second.  The
Intersect operator produces unique rows form the first result set that are in the second.  

#### Columnwise ####
The Union operator aligns columns by their position in both result sets.  The Outer Union operator produces all columns 
from both result sets.  The Except operator aligns columns by their position in both result sets.  The Intersect aligns columns
by their position in both result sets.  

#### Other Syntax ####
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

#### Displaying Query Results ####
You can change what a column is called in the output by adding a `'label'` after it in the `SELECT` statement.  The label
can be up to 256 characters of text.  You can also call this with `LABEL='label'`.    

To alter the format of a `PROC SQL` step, you can add the `FORMAT=formatw.d` statement in the `SELECT` statement.

You can add titles and footnotes to output using the `TITLE<n>'text'` and `FOOTNOTES<n>'text'` statements.

Note also that you can add a constant (numeric or string literal) to `PROC SQL` output using the syntax `'constant-text' <AS alias>`.

#### Summary Statistics ####
Both ANSI-standard and SAS SQL contain a number of functions for summarizing data.  These include `MAX`, `MIN`, `SUM`, etc.
If you pass one argument to most SAS SQl functions, they will calculate down a column; If you pass two arguments, they will calculate rowwise. Most 
ANSI SQL functions can accept only a single argument.

You can count the number of rows using the ANSI standard `COUNT()` or `SUM()` functions
or the SAS `_N_` function. The count function counts the numbers rows in a table or subset that have a nonmissing value for the column so
long as an argument is applied.  If you supply * as an argument, the function returns the total number of rows in a table or subset. The `FREQ` and `COUNT` functions
can not accept * as an argument.

Note that when combining summary statistics and normal data in a table, the summary statistic will be remerged with the table (i.e. every row will have the same value for the summary statistic).

The `GROUP BY` expression can be used to summarize data by a specific column.

You can use the position of a column in the select statement in the `ORDER BY` statement to shorten how much you have to type (e.g. `ORDER BY 2, 1, 3`).

#### Comparison to Data Step ####
Note that the data step, passed with multiple set statements, can also be used to combine data sets vertically.  

#### Having Clause ####
The `HAVING` clause acts similarly to the `WHERE` clause in that it subsets data, except the syntax rules are a little different.  You are allowed
to use the column aliases of calculated columns without using the keyword `calculated`.  In fact, the `WHERE` statement can't use calculated columns at all, while the
`HAVING` clause can.

#### Find Function ####
You can use the `FIND` function to identify rows that meet a criterion. This function searches a string for a specific substring and returns an integer that represents
the starting position of the substring within the string.  The full syntax for this is `FIND(string, substring <,modifier(s)><,startpos>)`.
The modifier `i` tells `FIND` to ignore case; the modifier `t` tells `FIND` to trim trailing blanks before searching.

The Find function is also useful as part of a Boolean expression.

#### Creating Tables and Views ####
You can create tables by:

1. Copying columns and rows from existing table(s)

2. Copy only the column structure from an existing table

3. Define new columns to create an empty table

These all require the `CREATE TABLE table-name`.  Each `CREATE TABLE` method can only create a single table. To create a table with the first method, you write
`CREATE TABLE table-name AS SELECT...;`.  To create an empty table that is structurally like another table, you use the `like` statement after `Create Table` (i.e. `Create Table table-name LIKE table-name`)

To create a totally new table, define the columns as:

```
CREATE TABLE table-name
	(column-name type <column-modifiers>
	<,...column-name type <column-modifier(s)>>);
```

Columns can be numeric or character (or any other ANSI-standard type), and modifiers include length, format, informat and label. 

The `INSERT INTO` statement adds data to tables.  The syntax is;

```
INSERT INTO table-name
	SET column-name=value,
			column-name=value, ... ;
```

If you want to add multiple rows, you use multiple `SET` statements.  

The `VALUES(value, value, ...);` statement adds values to the column
in every row. 

```
INSERT INTO table-name
	<(column<, ... column<)>
	VALUES(value, value, ...)
	<VALUES(value, value, ...)>;
```

If a column list is provided, the order of values in the value statement must match it.  If not, it must match the layout of the table.


### Views ###

A `PROC SQL VIEW` is a stored query, or virtual table, with a a name that can be referenced similar to a normal table.  Whenever
a view is referenced, the stored query is run and produces a temporary table with the most recent data.  Per ANSI standards, a view needs to
reside in the same library as its source(s).

Views are good because they allow you to avoid storing copies of large tables, avoid a frequent refresh of table copies (views always surface
the most current data), combine data from multiple tables, simplify complex queries, prevent other users from inadvertently altering the query code and
prevent other users from viewing data that they should not be able to see.

However, Views might produce different results each time they are accessed if the data in the underlying data sources changes.  They
can require significant resources (CPU and memory) each time they execute.  As well, if you are accessing data multiple times in the same program,
it might be better to create a table to ensure that the same data is returned each time.

The syntax to create a view is 

```
CREATE VIEW proc-sql-view AS
SELECT ...;
```

In a `CREATE VIEW` statement, the implicit libref is **not the work library**.  The implicit libref is whatever library is referenced in the first line of the statement
because ANSI standards require views to be in the same libraries as their source data.  

The `USING` clause in the	`CREATE VIEW` statement allows you to create views that are stored in a different physical location than its source tables.

```
CREATE VIEW proc-sql-view AS
SELECT ...
	<USING Libname-clause<, ...LIBNAME-clause>;
```

Note that the librefs in the `USING` statements are local to the view so you can call assign the same name to that libref that you do to the libref
in your `CREATE VIEW` statement.

#### Dictionaries ####

Dictionary tables or views contain metadata about each SAS session or batch job.  They are read-only.  SAS automatically assigns the libref
`dictionary` to all dictionary tables or views.  These are only available from within `PROC SQL`.  

SAS provides views in the `sashelp` library that are similar to dictionary views.  They can be used in other `PROC` or `DATA` steps.

To prepare for working with dictionary views, you should use the `DESCRIBE TABLE(table-1<, ...table-n>)` statement to see what you're looking with.
This sits inside a `PROC SQL` step.

#### Using the Macro Language with PROC SQL ####

The SAS macro language is a text processing tool that allows you to automate and customize SAS code.  When macro variables
are included as part of a `PROC SQL` step, SAS substitutes the appropriate values into the query before executing.

Macro variables can supply complete or partial SAS steps or statements.  Automatic macro variables are stored in an area of memory called 
the **Global Symbol Table**.  All variable names are stored in the global symbol table in uppercase letters. 
To display the contents of this, write `%put _all_`; to display all automatic macro variables, write `%put _automatic_;`

Use `%LET variable=value;` to create a macro variable.  **All macro variables are stored as text strings**.  SAS removes leading and trailing blanks from the
value of a macro variable but does not remove blanks within a value.

You can create macro variables from a query with the syntax:

```
SELECT column-1 <, ...column-n>
	INTO :macro-variable-1<,...:macro-variable=n>
	FROM table|view;
```

When referencing a macro variable from within a text string, you must enclose the text string within double quotation marks.

