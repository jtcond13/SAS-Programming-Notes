##SAS Notes: Part 2##

Typically, in `DATA` steps, SAS reads in one observation and reads out another.  However, it's possible
to control this using additional statements in one's code.  The traditional manner is to use the `set` statement
in a `DATA` step.  In this method, there is an implicit `OUTPUT` statement after each `SET` statement and an implicit
`RETURN` statement after each row of observations is read into the PDV.

To alter the data set produced in the `DATA` step, an `output;` statement is used between lines of code
to read the current contents of the PDV into the new data set.  This can be used to add computed values to an ouput set.
When SAS finishes reading an observation from the PDV into the new data set, all variables from the set statement contain their same value while variables created with assignment
statements are set back to zero.  

You can also use the `DATA` step to create multiple data sets from one using conditional statements. The syntax for this and an example of this is below.

```
IF expression THEN
	executable statements
ELSE IF expression THEN
	executable statements
ELSE executable statements;

data usa australia other;
	set orion.employee_addresses;
	if Country='AU' then
		output australia;
	else if Country='US' then
		output usa;
	else output other;
run;
```

Sometimes, it can be more efficient to use a `SELECT` statement than a series of `IF..THEN` statements.  An example of this syntax 
is below.

```
SELECT (expression);
	WHEN-1 (when-expression-1, when-expression-2, ...) statement;
	WHEN-2 (when-expression-1, when-expression-2, ...) statement;
	OTHERWISE statement;
END;
```

The select expression is any SAS statement that evaluates to a single value. The otherwise statement specifies what logic to follow if no conditions are true. 

You can further subset expressions using the SELECT expression by adding a `DO` loop after a when statement.

In `DATA` step processing, you can use `DROP` and `KEEP` statements to determine whether a variable should be read into the output data set.
For example, in the below data set, the variable Street would be dropped in the output data set.

```
data usa australia other;
   drop Street_ID;
   set orion.employee_addresses;
   if Country='US' then output usa;
   else if Country='AU' then output australia;
   else output other;
run;
```

These statements can also be used in the same line as the `DATA` or `SET` statements.  The syntax is 
`Sas-data-set(drop=variable)` or `Sas-data-set(keep=variable)`.  When the `DROP` or `KEEP` statements are used in the set statements,
variables that are not selected are not read into the PDV (i.e. not available for processing).  However, when they are used in the data statement, they are.

You can also limit the number of observations that a `DATA` step reads in.  By default, SAS will process the entire data set.  You 
can use the `FIRSTOBS` and `OBS` statements to determine where SAS starts and finishes, respectively. The syntax for this is `SAS-Data-Set(obs=n)`.

##Summarizing Data##

An **accumulating variable** accumulates the value of another variable and keeps its own value from one observation to the next.  To create
an accumulating variable, you must use a `RETAIN` statement, the syntax of which is `RETAIN variable-name <initial-value>`.  Initial value
option defaults to a missing value before the first execution of the `DATA` step.  In the example below, the Mth2Dte variable is initialized
with a value of 50.

```
data mnthtot;
	set orion.aprsales;
	retain mth2Dte 50;
	Mth2Dte=Mth2Dte+SaleAmt;
run;
```

Problems can arise when a missing value is operated on by a retained variable.  This is because operation on a missing value returns a missing value.
To get around this, you can use the **sum statement (+)** instead of the combination of the `RETAIN` and assignment statements.

###By-Group Processing with Accumulating Variables###

The `PROC Sort` step can be used to sort data, with the `BY` statement used to choose the variable to sort on. Then you can add a `BY` statement to the `DATA` Step to process the data in groups. 

The syntax is:

```
Data output-SAS-data-set;
	SET input-SAS-data-set;
	BY BY-variable...;
	<additional SAS statements>
RUN;
```

To recognize the first and last observations in a group, you can use the temporary variables **First** and **Last**. For example, to accumulate the 
salaries of employees in each departments (or dept.), the syntax is:

```
data deptsales;
	set salsort;
	by Dept;
	if First.Dept then DeptSal=0;
	DeptSal+Salary;
	if Last.Dept;
run;
```

To initialize a new **accumulating variable** within a group, use a `DO...END` loop after a subsetting `IF` statement.   For example:

```
data output-data-set;
	set input-data-set;
	if First.variable = 0
		then do;
			Total_Variable = 0;
		end;
	Total_Variable + Variable
	if Last.variable = 1;
run;
```

In this example, the final if statement makes sure that the value is ouput only when the group is finished processing.

The same operation is done when there are multiple `BY` variables.  SAS creates `First.` and `Last.` variables for each variable.  When
the last observation of the primary variable is resached, SAS sets the `Last.` variable to 1 for the primary and all subsequent by variables.

###Reading Raw Data###

The `INPUT` statement describes the arrangement of values in the input data record.  **List input** can read standard or nonstandard data separated by a delimiter.  
**Column input** can read standard data types that is already arranged in columns.  **Column input** specifies the exact starting and ending column for each variable. An example is:

The syntax for the `INPUT` statement is:

```
Data SAS-data-set;
	INFILE 'raw-data-file-name';
	INPUT specifications;
```

**Standard data** is anything that SAS can read without any special instructions.  This includes whole numbers, decimals, +/- signs, and scientific or e notation.
**Nonstandard data** includes special characters ($, %, etc.), commas, date values, hexadecimal and binary.  

The `INPUT` statement for **standard** data is `INPUT variable startcol-endcol...;`.  The `INPUT` statement for **nonstandard** data is `INPUT column-pointer-control variable informat`.  **Column-pointer-control** moves SAS to the
start of the input variable, **variable** names the variable and **informat** specifies how SAS reads a variable (e.g. `$w.` or `DOLLARw.d`).

Two common pointer controls are `@n` and `+n`.  `@n` is an absolute reference, and moves the pointer to column n.  The `+n` pointer moves the pointer n columns to the right from
the end of the last variable that was read in.  The default pointer control is column one. 

In this operation, the program data vector is populated from the input buffer, where SAS stores a value in each column. Note that 
numeric variables are automatically stored in eight bytes of storage but you have to specify an informat so that the number gets sufficient
column space.  

###Creating a Single Observation from Multiple Records###

To create a single observation from multiple rows in an input text file, you can use multiple input statements.  These
input statements would omit pointer controls and thus each row would be read into the input buffer on its own.  

You can also use line pointer controls to avoid writing multiple input statements.  The forward slash (/) is a relative line pointer control, and moves the pointer relative to the line on which it is currently positioned.
For example "/ /" would move the pointer control two rows ahead.  The pound n (#n) pointer control is an absolute pointer, sending the pointer control n lines ahead of its current place.  

The syntax for these is `INPUT @n/+n variable informat`.

###Controlling When a Record Loads###

To stop an input statement from moving to the next record in a raw data file, you can use a line hold specifier to keep the current record in the input buffer.
The syntax for this is `INPUT ..... @`.  This holds the record in the input buffer until an input statement without a line hold specifier is read (`@`).
This enables you to nest input statements within conditional logic (if... else if) statements.  It's best to place conditional logic right after the place in an input statement where 
the variable being tested is assigned an informat (and thus read into the input buffer).

###Manipulating Character Values###

**SAS functions** performs a transformation on, or performs a calculation from, the arguments supplied to the function. The syntax is:
`function-name(argument-1, argument-n)`.  Arguments to SAS functions can be **constants**, **variables**, **SAS Expressions** or **other functions**.

A **target variable** is defined as a variable whose type, length and value are determined by the results of a function.  **Character functions** return values of different lengths.

One important character function is the `SUBSTR` function.  The syntax for assigning the results of the substring function to a variable is:
`var=SUBSTR(string, start,<length>)`.  The string argument can be a character constant, variable or expression. The start argument specifies the starting position (SAS is not zero-indexed, first position is 1, not 0).
The length argument specifies the number of characters to extract.  When `SUBSTR` is used on the left side of an assignement statement, it replaces variable values.  

Another important character function is `LENGTH(argument)`.  The length function returns the length of a character string, excluding trailing blanks.  Blank values return a value of 1.  
This is very useful, in combination with `SUBSTR`, to extract a string.  Other character functions include:

1) `RIGHT`, which right-aligns a character value and brings trailing blanks to the beginning of the value

2) `LEFT`, which left-aligns a character value and brings trailing blanks to the end of the value

3) `CHAR(string, position)`, which returns a single character from a specified position in a string

4) `PROPCASE(argument,<delimiter(s)>)`, which converts all letters in a value to proper case; any character used to separate words (including space) must be indicdated in the second argument as a string

5) `LOWCASE` function converts all characters to upper case, does not require specifying a delimiter

6) `SCAN(string, n, <delimiter(s)>)` returns the nth word from a character string

7) `CATX` removes leading and trailing blanks, inserts delimiters, and returns a character string

8) `TRIM` Removes trailing blanks from a character string

9) `STRIP` Returns a character string wit all leading and trailing blanks removed

10) `CAT`, `CATT`, `CATS` return concatenated character strings; `CAT` does not remove any blanks, `CATT` removes trailing blanks and `CATS` strips leading and trailing blanks

11) `CATX(separator, string-1,..., string-n)` removes leading and trailing blanks, inserts separators and returns the concatenated string

12) `FIND(string, substring, <modifiers,start>)` searches for a specific substring of characters within a character string and returns an integer that represents the starting position of the substring

14) `TRANWRD(source, target, replacement)` replaces or removes all occurrences of target substring in source string

15) `PROPCASE` converts all letters in a value to proper case

16) `COMPRESS(source, <chars>)` removes specified characters from a string, if no characters are specified, removes blanks 

Other things to note are:

-In string functions, the delimiter argument should be a string that contains all possible delimiters.  The string ', ' would indicate that commas or spaces are delimiters, for example.

-You use the substring function when you know the order of all characters in a value. You use the scan function when you know the order of the words in a value, but not the order of characters.

-The concatenation operator is !!.  The syntax to create a new variable from concatenated strings is `NewVar=string-1 !! string-2;`

##Numeric Functions##
Some important descriptive statistic functions are:

1)`SUM` returns the sum of the nonmissing arguments

2) `MEAN` returns the average of the arguments

3) `MIN` returns the smallest value from the arguments

4) `MAX` returns the largest value from the arguments

5) `N` returns the number of nonmissing arguments

6) `NMISS` returns the number of nonmissnig numeric arguments

7) `CMISS` returns the number of missing numeric or character arguments


One way to write SAS numeric functions efficiently is to use a **variable list** as an argument,
rather than specifying a list of variables or values individually.  This is accomplished with the keyword **of**.  
One example is:

`Avg = Mean(of Qtr1-Qtr4);`

Some types of **variable lists** are:

-Numbered range: all variables x1 to xn, inclusive (the example above would be of this type)
-Name range: all variables ordered as they are in the program data vector, from x to a inclusive
an example of this would be:

`Avg = Mean(of Qtr4--Fifth);`

You can also use a name range to specify all variables of a certain type, e.g.:

`Avg = Mean(of x-numeric-a);` or
`Avg = Mean(of x-character-a);`

As well, you can specify all variables that begin with the same string.

`Avg = mean(of Tot:);`

Spacial SAS name lists allow you to specify all of the variables, all of the character variables
or all of the numeric variables that are already defined in the current DATA step.

`Avg = mean(of _All_);` would take the mean of all variables currently defined in the data step.
`Avg = mean(of _Numeric_);` would take the mean of all numeric variables.

###Rounding Values###

- `ROUND(argument, <round-off-unit>);` A value rounded to the nearest multiple of the round-off unit(defaults to nearest integer)

- `CEIL` The smallest integer greater than or equal to the argument

- `FLOOR` returns the largest integer that is less than or equal to the argument

- `INT` returns the integer portion of the argument (truncates the decimal portion)

##Character and Numeric Conversion##

If you reference a **character variable** in a **numeric context** (such as in an arithmetic operation), SAS 
tries to convert the values to numeric variables.  You can also convert automatically by **assigning character values to
a previously designated numeric variable**.  Automatic conversion produces a missing value for any character value
that does not conform to standard numeric form (w.).  

To explicitly convert **character values to numeric values**, you can use the `Input(source, informat)` function. This converts the value in source
to the informat specified in the second argument of the function.

SAS automatically converts numeric values to character when you use the concatenation operator(or any `CAT` function).  When SAS automatically converts
numeric values to character, it uses the **Bestw.** informat, which will add leading blanks to numeric values fewer than 12 digits
long.

To explicitly convert **numeric values to character values**, you can use the `Put(source, format)` function. This converts the value in source
to the format specified in the second argument of the function. 

Note that you can not convert a variable by using the syntax `variable = input(variable, informat)` but instead need
to rename the variable and run `INPUT` or `PUT`.


##Debugging Techniques##

**Syntax Errors** occur when a program does not conform to the rules of the SAS language.  When this happens,
SAS writes a message to the SAS log.  A **logic error** is when a program follows syntax rules, but the results
aren't correct.

One way to display variables messages and values is with the `PUTLOG <specifications>` statement. For example,
for variable **name**, the statment `PUTLOG name=;" would right the value of the name at that point in the program 
to the log. You can specifiy a format for the output with the syntax `variable=format` to ensure that the variable is printed correctly.
`PUTLOG _all_` will write the value of all variables to the log.  

One variable you may want to write to the log is the automatic variable **_ERROR_**. **_ERROR_** is an automatic
variable that SAS creates with every data step, and it receives a value of one if the data step encounters input data errors
, conversion errors, math errors or certain other errors.  

Another important automatic variable is the **_N_**, which increments at every loop of the data step.  You may recall this
from various procedures that have been covered, as it is also the variable which counts observations in a data set.

You can use the value of **_N_** to apply conditional logic, e.g. :

``` 
if _n_1 = 1 then
putlog 'First Iteration';
```

You can also use the `end = variable` option as part of a `SET` or `INPUT` statement, to create a variable that will equal
one when the data step reaches its final iteration.


##Iterative Do Loops##

A `DO` loop is analagous to a **for loop** in most languages.  The syntax for it is:

```
Do *index-variable=start* to *stop* <by *increment*>;
	iterated SAS statements...
End;
```

The *start*, *stop* and *increment* statements must be numbers or expressions that evaluate to numbers.
If you omit the `BY` statement, the increment defaults to 1.

The loop executes until the value of the index variable **exceeds** the value specified in the stop statement.
When the starting value of your index variable is greater than the stopping value, you must use a negative number
for the increment.

Alternatively, you can use an item list instead of an index so that the code within the do loop executes when any of the 
statements in the item list are true.  The syntax and an example are below.

```
Do index-variable=item-1, <...item-n>;
	iterated SAS statements...
end;

data investments;
	do year=2010, 2011, 2012;
		capital+500;
		capital+(capital*.045);
	end;
run;
```

By default, SAS will only output to a dataset the values of variables in the last iteration of the loop.
If you add `output;` to the end of the code nested in the do loop, SAS will write the contents of the PDV out at the end of every
iteration of the loop.

You can also have a loop execute until a condition evaluates to true.  SAS checks if the expression
is true after executing the code inside the loop (do...until always executes at least once).  The syntax is:

```
Do Until (expression);
	*iterated SAS statements...*
End;
```

Conversely, you can execute a `Do` loop while a condition evaluates to true.  In this case, SAS checks if the expression
is true before executing the code inside the loop.  The syntax is:

```
Do While (expression);
	*iterated SAS statements...*
End;
```

You can add a conditional clause to a do loop by including both an index variable and an until or while statement.
The syntax for this is:

```
Do *index-variable=start* to *stop* <by>
	Until | While (expression);
	iterated SAS statements...
End;
```

`Do` loops can be nested, but you must use different index variables.

##SAS Arrays##

A SAS array is a temporary grouping of variables (all of the same type), assigned to a unique name, that exists for the duration
of the data step. SAS arrays are not data structures.

An array is created with the syntax:

```
ARRAY array-name {dimension}<array-elements>;
```

In this statement, dimension is the number of variables being grouped.  You can also use the special SAS names `_numeric_` or `_character_`
to group all numeric or character variables together.  You can use an array subscript `array-name{subscript}` to reference a variable within
an array.

You can use the subscript notation to loop over elements in an array.  The syntax for this is:

```
DO index-variable=1 to number-of-array-elements;
	<additional SAS statements>
END;
```

In this situation, you would use the index variable i to loop through the array elements.

Another useful trick is the `DIM` function.  The `DIM(array-name)` function returns the number of elements
in the array.

Arrays can be passed to a function like a variable list you must include the word `of` when you do this, however.
Also, you can create variables with an array, listing the values of variables after the array statement.

You can create an array with inital values (For calculation) by adding a list of values, separated by commas and enclosed in parentheses, at the end of an array statement.
To indicate that this array will not be needed in the output, you can use the SAS statement `_temporary_`.  This is useful when you only need the array to perform a calculation
or look up a value.  Arrays that are marked `_temporary_` can not be referenced directly, but by their subsetting number.  This is because they are not created in the PDV, but instead
held in a separate area of memory.  

##Match Merge Functions##

In a data step, you can create a new data set that is the result of merging two other data sets.
You do this with the `merge` statement.  Before merging, you must sort (`PROC SORT`) on the merge variable before you can 
merge.  

When you match merge data sets with same-named variables, the data step overwrites values in one of the data sets with values
from the other data set.  The best way to avoid this isto use the `Rename=` option in the data step.

##Creating Formats##

The `PROC FORMAT` procedure can be deployed to create formats.  The syntax for this is:

```
PROC FORMAT;
	VALUE format-name value-or-range1='formatted-value1'
										value-or-range2='formatted-value2'
										...;
RUN;
```

User-created formats, like SAS formats, must have a period after them when referred to in code.

The syntax to create a format from another data table is:

```
PROC FORMAT CNTLIN=SAS-data-set;
RUN
```

The control data set, however, must contain variables **FmtName**, **Start** and **Label**.  If the format specifies a range, the data must also
contain the variable **End**.  The dataset must also contain the variable **Type** for character variables, unless the FmtName variable beings with a dollar sign.

SAS formats are stored in a special type of directory called a **catalog**.  These contain a **.formats** extension.

Format statements can be referenced in `FORMAT` statements, `FORMAT=` options, `PUT` statements, `PUT` functions and `WHERE` or `IF` statements.

To specify the order in which SAS looks for formats, you can use the `FMTSEARCH= (location1 location2 location3)` option.
Sometimes, you'll try to load a format that SAS can't load.  In that case, using the `OPTIONS NOFMTERR;` system option will default these 
to $w. or w. formats.

SAS searches in the order specified in the FMTSEARCH= option. By default, SAS searches in the work and library libraries first unless you specify them in the option. 

You can nest formats in `PROC FORMAT` statements, assigning them to values the same way you would other formats.  They should, however,
be enclosed in brackets.  `PROC FORMAT` can also be used to document the formats in a catalog.  Use 

For managing formats, the `PROC CATALOG` statement can be invaluable. For more details, see documentation.

To update a format whose original code you don't have, you start by using `PROC FORMAT` to create an output data set.  The syntax for this is:

```
PROC FORMAT LIBRARY=libref.catalog
						CNTLOUT=SAS-data-set;
```

You can then alter that data set using standard techniques and then recreate a format using:

```
PROC FORMAT LIBRARY=libref.catalog
						CNTLIN=SAS-data-set;
```

##PROC SQL##

`PROC SQL` allows you to write ANSI-Standard SQL in SAS programs.  `PROC SQL` can access SAS data sets, like other SAS procedures, but can also access data in
other database engines.  `PROC SQL` allows you to retrieve and manipulate **tables**, which can either refer to SAS datasets or RDBMS tables.

`PROC SQL` can join tables and produce a report in one step, rather than creating a SAS data set in a `DATA` step.  SQL also allows you to join tables
without pre-sorting the input.  On the other hand, the `DATA` step allows you to create multiple data sets, direct output to data sets based on which input data
set contributed to the observation and perform complex manipulation (do loop, `First.` and `Last.` processing, arrays and the Hash object).

As the subject of these notes is SAS, I won't enumerate here the various parts of a SQL query: *SELECT*, *FROM*, *WHERE*, etc.

##SAS Macro Facility##

The SAS Macro Facility is a utility for extending and improving upon SAS to limit the amount of code you must write to complete a task.

**Macro** variables substitute text into SAS programs, **Macro programs** generate custom SAS code. 

Certain macro variables are created by the SAS system but you can also create your own.  Automatic macro variables include:

-&SYSDATE9

-&SYSDAY

-&SYSTIME

The syntax to assign a macro variable is `%LET macro-variable=value`. SAS does not evaluate mathematical expressions as all macro variables are text strings.

Macro variables are referenced with an ampersand (&).  For example, a macro variable defined as `%LET jack=23` would be referenced as `&jack` in the sas program.

The system option `SYMBOLGEN` (`NOSYMBOLGEN`) controls whether macro variables are written to the SAS log.  Defaults to `NOSYMBOLGEN`.  
Another way of writing messages to the log is using the `%PUT` statement.  SAS resolves macro variables before writing the message to the log.

