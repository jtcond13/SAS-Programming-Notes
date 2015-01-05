SAS Programming Introduction
=========================
SAS programs are made up of steps. Steps are made up of statements. Each step begins with PROC or DATA, ends with:
	run;

Each statement begins with a keyword and ends with a semi-colon

Three types of steps:
	*Data Steps: Reads data from an input source and creates a SAS data set (.sas7bdat)
	*Proc Steps: Processes a Data set (generates reports, graphs, sorts data, set)
	*Global Statements: These affect the outcome of both Data and Proc steps
	

 
SAS Data Sets:
	* Columns are variables
	* Rows are observations
	
Syntax Rules
==================
	*Whitespace doesn't matter (free-format)
	*Reserved words are not case-sensitive
	*Block comments formatting:
		/*this is a block comment  */
	*Line comment formatting:
		*this is a line comment;
	*Programs are executed one step at a time, steps found to have syntax erros are not executed 
	*Syntax errors will appear in the log
	