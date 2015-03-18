SAS Programming Introduction
=========================
SAS programs are made up of **steps**. **Steps** are made up of **statements**. Each step begins with `PROC` or `DATA`, and should end with `run;`

Each statement begins with a keyword and ends with a semi-colon

**There are three types of steps:**

1. **Data Steps**: Reads data from an input source and creates a SAS data set (`.sas7bdat`)

2. **Proc Steps**: Processes a Data set (generates reports, graphs, sorts data, set)

3. **Global Statements**: These affect the outcome of both `Data` and `Proc` steps

 
	
Syntax Rules
==================

- Whitespace doesn't matter (free-format)

- Reserved words are not case-sensitive

- Block comments formatting:
	`/*this is a block comment  */`

- Line comment formatting:
	`*this is a line comment;`

- Programs are executed one step at a time, steps found to have syntax erros are not executed 

- Syntax errors will appear in the log

- Variable names are not case sensitive; they must be 1-32 characters and start with a letter or underscore, followed by letters, underscores and numbers

	
Data
===========
- Data is stored in libraries, which are referred to in code as `LIBREF`

- You can create your own libraries using the `LIBNAME` assignment statement.  Some systems will do this via a Macro.  

- Data Sets contain a table, composed of variables (columns) and rows (observations), and a descriptor portion, composed of information about the data set and variable attributes such as name, type (character or numeric) and length.

- `PROC CONTENTS DATA=libref.SAS-data-set; RUN;` will display the data set descriptor portion

- Missing value are valid in SAS, with missing character values just showing ' ' and missing numeric values displayed as '.'

- Use `PROC CONTENTS` with `libref._ALL_` to display the contents of a SAS library. The report will list all the SAS files contained in the library, as well as the descriptor portion of each data set in the library.

- You'd access a library like:

	%let path=**filepath**;
	libname orion "&path";

- The SORT procedure is accessed as follows: 

		PROC SORT DATA=input-SAS-data-set
                      <OUT=ouput-SAS-data-set>;
         BY <DESCENDING> by-variable(s);
		RUN;

- Data can be subset in Print PROC through:

	PROC PRINT DATA=SAS-data-set;
        VAR variable(s);
        SUM variable(s);
	RUN;


Formatting Output
===========

- `Titlen'text'	 and `Footnotesn'text'` add footnotes; the number n (1 - 9) indicates length from the top to the bottom 

- `Lable variable='label'` assigns a label to a variable, in case you don't want the variable name to appear in the report

- `split='*'` tells SAS to wrap a variable name or label (i.e. across two lines) 

- `FORMAT variable(s) format;` can be added to `PROC Print` in order to change how variables are formatted in reports

- There are a variety of SAS formats available for both character and numeric formats.  Character formats begin with a dollar sign; dollar formats don't.

- You create your own formats with the Format procedure, defined as:

	PROC FORMAT;
        VALUE format-name value-or-range1='formatted-value1'
                                           value-or-range2='formatted-value2'
                                           ...;
	RUN;
	
- The name of a character format must begin with a dollar sign, followed by a letter or underscore, followed by letters, numbers, and underscores. Names for numeric formats must begin with a letter or underscore, followed by letters, numbers, and underscores. A format name cannot end in a number and cannot be the name of a SAS format.

- When you define a numeric format, it is often convenient to use numeric ranges in the value-range sets. Ranges are inclusive by default. To exclude the endpoints, use a less-than symbol after the low end of the range or before the high end.

- The LOW and HIGH keywords are used to define a continuous range when the lowest and highest values are not known. Remember that for character values, the LOW keyword treats missing values as the lowest possible values. However, for numeric values, LOW does not include missing values. 

Reading Data in and Out
==============================

- The SAS data step reads data in.  It is written: 

```
	DATA output-data-set;
		SET input-data-set;
		WHERE where-expression;
	RUN;
```

- SAS date constants allow you to convert SAS dale values (which you'll recall are numbers indicating the distance from Jan 1, 1960)
to dates.  They are written `'ddmmm<yy>yy>'D`

- You can create or change the value of a variable with the `variable=expression`.
The variable can be a new or existing variable and the expression is anything that produces a value.

- An expression is thus some combination of operands (constants and/or variables) and operators (arithmetic calculation symbols or SAS functions)
. For arithmetic symbols, note that SAS respects the order of operations (i.e. PEMDAS).  However, all operations involving a missing value return a missing value.

- Variables can be included or excluded from an ouput data set with the DROP and KEEP statements.
These are written `DROP variable-list;` and `KEEP variable-list;`.

- SAS proccesses data steps in two steps:

###Compilation Phase###
1. Scanning the entire program for syntax errors.
2. If the program is executable, SAS compiles the program to machine code.
3. SAS creates the Program Data Vector (PDV) to hold all observations. This is an area of memory 
where SAS builds an observation.  It also contains two automatic variables `_n_` (iteration number) and `_ERROR_` (a categorical variable indicating an error) 
4. SAS creates the descrptor portion of the new data set.
  
###Execution Phase###
1. SAS reads and processes observations, creating the data portion of the output.

- To subset a data set based upon variables you create, you can use an `if` statement.  

- `If` statements can not be used in `PROC` steps; `WHERE` must be used instead.  `WHERE` in data steps can only reference variables in all input data sets.

- The `FORMAT variable(s) format;` statement can be used in a data step to permanently change the format of variables.

Accessing Non-SAS Data
=======================

- SAS/ACCESS `Libname` provides access to non-SAS data sources.  SAS/ACCESS engines are licensed separately.

- You can add the `INFILE` statement to a Data step to read a csv. This is similar to read.csv in R. This is written:

```
data ouput-data-set-name;
	infile "&path/file" dlm = " "
	input specifications;
run;
```

- Delimiter defaults to blank. 
	
- To access database data, the correct syntax is:

```
libname libref engine <user= "", password="", path="", schema"">
```

- To read raw data into SAS, the DATA step is used.  The syntax is as follows:

```
DATA ouput-data-set;
	INFILE "&path/raw-data-file-name" dlm='';
	INPUT specifications;
RUN;
```

- The delimiter defaults to a a blanK space so if you're using a .csv then you'd write `dlm = ","`

- The `INPUT` statement reads fields in the order they are in the raw file.  You name the variables how you'd like them and separate with `$`.

- During the compilation phase, SAS scans the data step for syntax errors.  SAS also creates an input buffer to read the file.  
Then SAS creates the Program Data Vector (containing _n_ (iteration counter) and _error_ (indicating error)).  The default 
length for all variables is 8 bytes.  

- To explicitly define the length of a character variable, you use a `LENGTH` statement.  This must proceed the INPUT statement.  The syntax is as follows:
```
LENGTH variable(s)<$>length;
```

SAS Functions & Creating New Variables
=======================================

- SAS Sum function accepts numeric constants, variables or arithmetic expressions (e.g. '2 + 3').  It is written:
`SUM(variable1, variable2, ...)`

- SAS Date function takes a SAS date (an integer representing distance from Jan 1, 1960) and returns something that can be used.
Examples includes:

```
YEAR(SAS-date)
QTR(SAS-date)
MONTH(SAS-date)
DAY(SAS-date)
WEEKDAY(SAS-date)
TODAY()
DATE()
```

- Conversely, the `MDY` function returns SAS date value from a set of inputs.  It's written:

```
MDY(month, date, year)
```

- New variables are assigned in the `DATA` step, with new variables defined as `variable=definition` 

- To combine data sets, you read in two data sets in one much as you would with one data set.  Data sets listed first will override sets listed later.  Syntax is:

```
Data Sas-Data-set;
	Set SAS-data-set1 Sas-data-set2 ...;
RUN;
```

- If two data sets contain different variables, you can rename the variables in one data set to match the first.  
	In the above set, that would look like `Sas-Data-set(RENAME(old-name-1=new-name-1))`
	
- Joining SAS data sets based on a common variable (one-to-one relationship) is called match-merging. 
	This is accomplished with a `Merge` statement embedded in a data step.  The syntax is:
	
```
data Output-data-set
	merge input-data-set-1 input-data-set-2
	by variable
run
```

- To combine data sets in a many-to-many fashion, the same syntax is used.

- When merging data sets, you can use the an IN statement to create a temporary variable 
	that indicates whether the associated data set contributed data to any current observation.
	This is created as a binary variable, with one indicating 'yes' and zero indicating 'no'.
	The syntax is:
		
```
data new-Sas-data-set;
merge Sas-data-set(in=variable);
by variable;
run;
```

- To produce a data set that only contains matches, you can use a subsetting `if`statement,
	as in:
	`if emps = 1 and mau = 1;`
	
	As Boolean variables, you can omit testing them for truth with an alternate syntax.  For example,
	instead of the above, you could simply write `if emps and mau`.  Instead of `if emps = 1 and mau = 0`,
	you could write `if emps and not mau`.

Control Flow
========================

- Conditional statements can be assigned as `IF expression THEN statement`, with expressions composed of constants and/or variables, operators or SAS functions
and statements being any executable SAS statement.


- SAS also support `ELSE` and `ELSE IF` statements as part of the control flow.  

- As well, there is a `DO; ... END;` loop that works much like a for loop in other languages. 

