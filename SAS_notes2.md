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
Column input can read standard data types that is already arranged in columns.  Column input specifies the exact starting and ending column for each variable. An example is:

The syntax for the `INPUT` statement is:

```
Data SAS-data-set;
	INFILE 'raw-data-file-name';
	INPUT specifications;
```

**Standard data** is anything that SAS can read without any special instructions.  This includes whole numbers, decimals, +/- signs, and scientific or e notation.
**Nonstandard data** includes special characters ($, %, etc.), commas, date values, hexadecimal and binary.  

The `INPUT` statement for **nonstandard** data is `INPUT column-pointer-control variable informat`.  **Column-pointer-control** moves SAS to the
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

###Controlling When a Record Loads###

To stop an input statement from moving to the next record in a raw data file, you can use a line hold specifier to keep the current record in the input buffer.
The syntax for this is `INPUT ..... @`.  This holds the record in the input buffer until SAS reaches the end of an input statement that doesn't have a line hold specifier (`@`).
This enables you to nest input statements within conditional logic (if... else if) statements.  It's best to place conditional logic right after the place in an input statement where 
the variable being tested is assigned an informat (and thus read into the input buffer).



