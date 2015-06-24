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

