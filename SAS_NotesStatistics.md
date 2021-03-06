#Statistics in SAS#

###Assessing a single variable###

- `PROC MEANS` calculates a standard set of descriptive statistics.  It's written:
```
PROC MEANS DATA=SAS-data-set <options>;
	CLASS variables;
	VAR variables;
RUN;
```

- The `CLASS` statement will group observations by the specified variables and display the number of missing values for each group.

- The options area is used to list what statistics you want. 

- `PROC UNIVARIATE` produces a set of plots and statistics to assess whether a univariate distribution is 
normal.  This is written:

```
PROC UNIVARIATE DATA=SAS-data-set <options>;
	VAR variables;
	ID variables;
	HISTOGRAM variables;
	PROBPLOT variables </ options>;
	INSET keywords </ options>;
RUN;
```

###Statistical Graphics in SAS###

- `PROC SGSCATTER` can produce a wide range of scatterplots.  The procedure is written:

- `PROC SGPLOT` can produce a wider variety of plots, including scatterplots, line graphs, histograms, regression lines, etc.
This procedure is written:
```
PROC SGPLOT DATA=SAS-data-set <options>;
	DOT category-variable </option(s)>;
	HBAR category-variable </option(s)>;
	VBAR category-variable </option(s)>;
	HBOX response-variable </option(s)>;
	VBOX response-variable </option(s)>;
	HISTOGRAM response-variable </option(s)>;
	SCATTER X=variable Y=variable </option(s)>;
	NEEDLE X=variable Y=numeric-variable </option(s)>;
	REG X=numeric-variable Y=numeric-variable </option(s)>;
RUN;
```

- An additional statement, `REFLINE`, can be used to add a reference line to the plot.



- `PROC SGPANEL` can produce plots for several different levels of a factor or several different time periods.

- `PROC SGRENDER` allows you to create plots from graph templates you have written yourself.

- Standard error measures variability of a sample statistic.
-  