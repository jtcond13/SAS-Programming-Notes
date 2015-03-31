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
