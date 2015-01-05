SAS Programming Introduction
=========================
SAS programs are made up of steps. Steps are made up of statements. Each step begins with PROC or DATA, and should end with `run;`

Each statement begins with a keyword and ends with a semi-colon

There are three types of steps:

*Data Steps: Reads data from an input source and creates a SAS data set (`.sas7bdat`)

*Proc Steps: Processes a Data set (generates reports, graphs, sorts data, set)

*Global Statements: These affect the outcome of both `Data` and `Proc` steps

 
	
Syntax Rules
==================
-Whitespace doesn't matter (free-format)

-Reserved words are not case-sensitive

-Block comments formatting:
	`/*this is a block comment  */`

-Line comment formatting:
	`*this is a line comment;`

-Programs are executed one step at a time, steps found to have syntax erros are not executed 

-Syntax errors will appear in the log

-Variable names are not case sensitive; they must be 1-32 characters and start with a letter or underscore, followed by letters, underscores and numbers

	
Data
===========
-Data is stored in libraries, which are referred to in code as `LIBREF`

-You can create your own libraries using the `LIBNAME` assignment statement

-Data Sets contain a table, composed of variables (columns) and rows (observations), and a descriptor portion, composed of information about the data set and variable attributes such as name, type (character or numeric) and length.

-`PROC CONTENTS DATA=libref.SAS-data-set; RUN;` will display the data set descriptor portion

-Missing value are valid in SAS, with missing character values just showing ' ' and missing numeric values displayed as '.'

-Use `PROC CONTENTS` with `libref._ALL_` to display the contents of a SAS library. The report will list all the SAS files contained in the library, as well as the descriptor portion of each data set in the library.

-You'd access a library like:

	%let path=**filepath**;
	libname orion "&path";
	