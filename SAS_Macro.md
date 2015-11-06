SAS Macro Facility
==================

The SAS Macro Facility is a text processing tool that allows you to substitute text in your SAS programs and automatically generate SAS code.

**Macro variables** substitute text into your SAS programs.  **Macro programs** generate custom SAS code. 

SAS programs are first processed by the **input stack**, which breaks the program into **tokens** and passes them to the **word scanner** one group at a time.
The **word scanner** then passes the statement to the appropriate compiler.  The following token types exist:

- **Name**: Contains a string of characters that begins with a letter or underscore and continues with underscores, letters or numerals.

- **Special**: Can be any character, or combination of characters, other than a letter, numeral or underscore (e.g. & . ** /).

- **Literal**: Contains a string of characters enclosed in single or double quotation marks. 

- **Number**: Integer and floating-point numbers, including SAS date constants.  

##SAS Functions##

`%SYSEVAL` can be used for evaluating arithmetic expressions; `%SYSEVALF` performs a similar task, but will coerce character variables to numeric
and return data in the `best32.` format. 

`%INDEX(source, string)` searches a source variable for a string and returns the position in the source where it begins.  

##SAS Call Routines##

SAS call routines act similarly to functions, except that they can not be used in assignment statements or expressions.  Rather, they can
alter the value of an argument.  One commmon routine is `CALL SYMPUTX(macro-variable, value);`, which changes the value of a macro variable.  

##Defining Macro Variables in PROC SQL##
Note that you can define macro variables using the `insert into:` syntax in `PROC SQL`.  For example:

```
proc sql;
select count(distinct customers) into:mem from table;
quit;
```

##Macro Programs##

SAS Macro programs allow the user to use control flow (e.g. `%IF...%THEN` and `%DO...%END`) and modularity (functions) into their SAS programs.

