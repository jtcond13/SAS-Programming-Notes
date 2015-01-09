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

- 