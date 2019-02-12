# 4. Update queries and built-in functions
## Connect to database
```SQL
USE ExampleDatabase;
```
## Basic update query
First of all a warning: **UPDATE queries will permanently overwrite data in a column!** Therefore, it is often best to create a new (empty) column first and to update this column, rather than overwriting existing data. UPDATE queries can be used with mathematical, text, date and logical functions. They can also add data from another table, which will be covered in the section on JOINS. The mathematical, text, date and logical functions shown in this section can also be used in SELECT queries. As shown in the previous section, SELECT queries do not change the exsiting tables; therefore, it can be helpful to run a SELECT query first, to check if the function creates the desired result and then to modify the syntax into an UPDATE query.

In it's most basic application, an UPDATE query can be used to copy data from one origin column to another target column. 
```SQL
ALTER TABLE questionnaire.recruitment --create new column
ADD smoker_2 int NULL; --same data type as original column
GO
UPDATE questionnaire.recruitment
	SET smoker_2=smoker; --target column = origin column
```
If the target column has the same data type as the origin column, the UPDATE function should run without problems. If the target column has a different data type than the origin column, SQL Server will try to convert the original data type to work with the target data type. This conversion is invisble to the user and is called an implicit conversion. This link https://docs.microsoft.com/en-us/sql/t-sql/data-types/data-type-conversion-database-engine?view=sql-server-2017 shows a matrix of the conversion between data types.

For more control, explicit data conversion can be used via the CAST or CONVERT function. In most cases CAST and CONVERT are interchangeable, i.e. they will produce the same result. Therefore, it often comes down to personal preference; however, one exception is dates. For coverting date or datetime columns, the CONVERT function is recommended, because it allows an output style to be specfied (see https://teamsql.io/blog/?p=1455 and https://docs.microsoft.com/en-us/sql/t-sql/functions/cast-and-convert-transact-sql?view=sql-server-2017).
```SQL
ALTER TABLE person.info
ADD dob_text varchar(10);
--CAST
UPDATE person.info
	SET dob_text=CAST(dob AS varchar(10)); 
```
CAST will convert the date into a string in  the same format as it is stored as a date. 
```SQL
ALTER TABLE person.info
ADD dob_date2 varchar(10);
--CONVERT
UPDATE person.info
	SET dob_date2=CONVERT(varchar(10),dob,103);
```
By adding style 103 to the CONVERT statement the string uses the dd/mm/yyyy output format. 

UPDATE can also be used with a where clause to selectively change rows: 
```SQL
ALTER TABLE person.info
ADD gender varchar(6);

UPDATE person.info
	SET gender='female'
WHERE sex=1;

UPDATE person.info
	SET gender='male'
WHERE sex=0;
```
## Mathematical operations
The arithmetic operators are:

|Operator|Use|
|---|---|
| + | addition |
| - | subtraction |
| * | multiplication |
| / | division |
| % | modulo |

```SQL
ALTER TABLE questionnaire.recruitment
ADD smokerPlusExercise int; 
GO
UPDATE questionnaire.recruitment
 SET smokerPlusExercise=smoker+exercise;
```
If one of the values in an arithmetic operation is missing (i.e. NULL), the result will be NULL.  
```SQL 
ALTER TABLE questionnaire.recruitment
ADD smokerPlusalcohol int; 
GO
UPDATE questionnaire.recruitment
 SET smokerPlusalcohol=smoker+alcohol;
```
Examples of the other mathematical operators using a SELECT query:  
```SQL
SELECT *
	,BP_systolic-BP_diastolic AS subtraction
	,BP_systolic*BP_diastolic AS multiplication
	,BP_systolic/BP_diastolic AS division
	,BP_systolic%BP_diastolic AS modulo
FROM health.bloodpressure
```
Note that the division results are all 1 or 2. This has happened, because both BP_systolic and BP_diastolic have data type int and therefore the resulting column is also an int. To get a result with decimals, one or both columns need to be converted into numeric data type first:  
```SQL
SELECT *
	,CAST(BP_systolic AS NUMERIC(4,1))/CONVERT(numeric(5,2),BP_diastolic) AS division_decimals
FROM health.bloodpressure
```
The result is shown with a greater number of decimal places than specified in the CAST or CONVERT statement. This link https://docs.microsoft.com/en-us/sql/t-sql/data-types/precision-scale-and-length-transact-sql?view=sql-server-2017 shows how the precision is set for different arithmetic operations. For divisions the minimum precision is 6 decimal places. 

## Mathematical functions
<table>
	<tr>
		<th>Function</th>
		<th>Meaning</th>
		<th>Example</th>
	</tr>
	<tr>
		<td>
			ABS 
		</td>
		<td>
			absolute value of a number
		</td>
		<td>
			<pre lang="sql" style="text-align: left;">
			SELECT*
				,ABS(temperature) as abs_temp
			FROM env.temperature;
			</pre>
		</td>
	</tr>
	<tr>
		<td>
			POWER 
		</td>
		<td>
			raises value to specified power 
		</td>
		<td>
			<pre lang="sql">
			SELECT*
				,POWER(temperature,3) as power_temp
			FROM env.temperature;
			SELECT*
				,POWER(concentration,0.5) as squareroot_conc 
			FROM env.pollutant
			</pre>
		</td>
	</tr>	
	<tr>
		<td>
			SQUARE 	
		</td>
		<td>
			square of a number
		</td>
		<td>
			<pre lang="sql">
			SELECT*
				SQUARE(concentration) as square_conc
			FROM env.pollutant	
			</pre>
		</td>
	</tr>
	<tr>
		<td>
			SQRT 		
		</td>
		<td>
			square root of a number
		</td>
		<td>
			<pre lang="sql">
			SELECT*
				SQRT(concentration) as squareroot_conc
			FROM env.pollutant
			</pre>
		</td>
	</tr>
	<tr>
		<td>
			EXP 
		</td>
		<td>
			exponential (raises value to base e)
		</td>
		<td>
			<pre lang="sql">
			SELECT*
				,EXP(concentration) AS exp_conc
			FROM env.pollutant
			</pre>
		</td>
	</tr>
	<tr>
		<td>
			LOG 
		</td>
		<td>
			logarithm of a number, base is optional, by default the natural logarithm will be calculated
		</td>
		<td>
			<pre lang="sql">
			SELECT*
				,LOG(concentration) AS ln_conc
				,LOG(concentration,5) AS logbase5_conc
			FROM env.pollutant
			</pre>
		</td>
	</tr>
	<tr>
		<td>
			LOG10	
		</td>
		<td>
			logarithm of a number with base 10	
		</td>
		<td>
			<pre lang="sql">
			SELECT*
				,LOG10(concentration) AS lg_conc
			FROM env.pollutant
			</pre>
		</td>
	</tr>
	<tr>
		<td>
			SIGN 
		</td>
		<td>
			returns -1,0 or +1
		</td>
		<td>
			<pre lang="sql">
			SELECT*
				,SIGN(temperature)
			FROM env.temperature	
			</pre>
		</td>
	</tr>
</table>


## Rounding
The ROUND function will round to a specified number of decimal places. If a negative number of decimal places is specified SQL will round to that number of places to the left of the decimal point, i.e. -1 will round to tens, -2 will round to hundreds and so on. The FLOOR and CEILING functions can be used to always round down or up. However, these always round to an integer.
```SQL 
SELECT *
	,ROUND(CAST(BP_systolic AS NUMERIC(4,1))/CONVERT(numeric(5,2),BP_diastolic),2) AS division_decimals
	,ROUND(BP_systolic,-1) AS negative_round
	,FLOOR(CAST(BP_systolic AS NUMERIC(4,1))/CONVERT(numeric(5,2),BP_diastolic)) AS rounddown
	,CEILING(CAST(BP_systolic AS NUMERIC(4,1))/CONVERT(numeric(5,2),BP_diastolic)) AS roundup
FROM health.bloodpressure
```
## Random numbers
The RAND function will return a pseudo-random number between 0 and 1.
```SQL
SELECT *
	,RAND(100) AS random
FROM person.info
```
However, every row in the random column, shows the same number, because the RAND function only ran once. To add a different random number in each, you could add a loop that runs RAND() multiple times or you could use the quick fix below. CRYPT_GEN_RANDOM() generates cryptographic randomly generated hexadecimal numbers. The number between the brackets is the length of the hexadecimal in bytes. The modulo is used to convert the hexadecimal into an integer. The random numbers generated range from 0 to n-1, where n is the number used in the modulo. 
```SQL
SELECT *
	,CRYPT_GEN_RANDOM(2)%100 AS random
FROM person.info
```
### Exercise
Add two columns to the env.temperature table. In the first column calculate the temperature in Kelvin. In the second column calculate the temperature in Fahrenheit.
<details>
	<summary>Click to see solution</summary>
	
```SQL 
ALTER TABLE env.temperature
ADD kelvin decimal(6,2) NULL
	,fahrenheit decimal(6,2) NULL;
GO
UPDATE env.temperature
	SET kelvin=temperature+273.15
		,fahrenheit=temperature*1.8+32;

SELECT *
FROM env.temperature
```

</details>

## String functions
### Concatenate
The CONCAT function can be used to concatenate two or more columns or user specified strings. The columns can have any data type; however, SQL will convert the data into strings, therefore the resulting columns will be a text column. If a user speficied string is used, it must be enclosed in single quotes. 
```SQL
ALTER TABLE questionnaire.recruitment
ADD fav_col_fruit varchar(20) NULL
	,fav_col_date varchar(20) NULL
	,smoker_alcohol varchar(3) NULL;
GO
UPDATE questionnaire.recruitment
	SET fav_col_fruit = CONCAT(fav_col,fav_fruit)
		,fav_col_date = CONCAT(fav_col,'2019-01-23')
		,smoker_alcohol = CONCAT(smoker,'_',alcohol);	 
```

### Length
The LEN function can be used to count the number of characters in a cell. It only works with text data type, not with numeric or date data types. It will count ALL characters, including white spaces.
```SQL
SELECT *
	,LEN(fav_col) AS length_colour
	,LEN(fav_fruit) AS length_fruit
	,LEN(fav_col_fruit) AS length_both
	,LEN(fav_col)+LEN(fav_fruit) AS length_check
	,LEN(smoker_alcohol) AS length_smok_alc
FROM questionnaire.recruitment;

SELECT LEN('1 2 3 4 5    ')--white spaces are counted
```
### Remove or replace characters
To remove leading or trailing white spaces from a string the LTRIM and RTRIM functions can be used. LTRIM will remove all white spaces to the left of the first character, RTRIM will remove all white spaces to the right of the last character. 
``SQL
SELECT LEN(LTRIM('   12345')) AS fromLeft;
SELECT LEN(RTRIM('12345   ')) AS fromRight;
```
SQL Server 2017 upwards also has the TRIM function. TRIM will remove white space from anywhere in a string. It can also be used to remove specific characters from a string. In SQL Server 2012, the same can be achived with the REPLACE function. The REPLCAE function uses the following syntax REPLACE('a string','characters to be replaced','new characters'). So to remove all white spaces you could use:
 ```SQL
 SELECT REPLACE('  1 2 3 4 5    ',' ','')
```
REPLACE can change any character pattern:
```SQL
SELECT*
	,REPLACE(fav_col,'bl','xx') as new_col
FROM questionnaire.recruitment;
```
### Extract part of a string
Useful functions to extract part of a string are LEFT,RIGHT, SUBSTRING and CHARINDEX. LEFT extracts the first n characteers from a string by using LEFT('a string', number of characters). The RIGHT syntax works in the same way, but extracts the last n characters from a string. The SUBSTRING function can extract from anywhere in a string by specifying a starting position and the number of characters after the starting position. CHARINDEX is useful when the starting position or the number of characters to be extracted varies. For example, the first part of a postcode can vary in length from 2 to 4 characters; therefore to extract the first part of a postcode, you could use CHARINDEX to find the white space separating the first and second part of the postcode and then use this with a LEFT function. The CHARINDEX syntax requires a character to be found, a string to be searched, and can optionally have a starting position: CHARINDEX('character to be found','a string to be searched','optional starting position'.
```SQL
SELECT *
	,LEFT(fav_col_fruit,5) as leftstring
	,RIGHT(fav_col_date,10) as rightstring -- the result shown is a still a string, not a date!
FROM questionnaire.recruitment;

SELECT*
	,LEN(postcode) as lengthPostcode
	,CHARINDEX(' ',postcode) as lengthFirstPart
	,LEFT(postcode,CHARINDEX(' ',postcode)-1) AS firstPart
	,SUBSTRING(postcode,CHARINDEX(' ',postcode)+1,LEN(postcode)-CHARINDEX(' ',postcode)) as secondPart
FROM person.info	 
```
For a list of all string functions go to: https://docs.microsoft.com/en-us/sql/t-sql/functions/string-functions-transact-sql?view=sql-server-2017

## Date functions
### Timestamp
SQL can add timestamps to data. The most commonly functions for this are SYSDATETIME and CURRENT_TIMESTAMP. They only differ in the number of fractional seconds they provide. Obvioulsy, these functions are most useful when adding rows to a table, but they can be used at any time. 
```SQL
SELECT*
	,SYSDATETIME() AS sysdate
	,CURRENT_TIMESTAMP as curr_timestamp
FROM person.info
```
### Adding or subtracting time
The DATEADD function can add or subtract timeperiods specified in a number of units ranging from nanoseconds to years. For a full list of available units see: https://docs.microsoft.com/en-us/sql/t-sql/functions/dateadd-transact-sql?view=sql-server-2017 . The function uses the following syntax: DATEADD('unit of timeperiod to be added, e.g. year or minute', 'number of timeperiods to be added', 'date to which timeperiod should be added').
```SQL
SELECT *
	,DATEADD(month,12,dob) as months12after
	,DATEADD(month,-12,dob) as months12before
FROM person.info;   

SELECT *
	,DATEADD(minute,10,vDateTime) as plus10minutes
FROM env.pollutant;
```
### Calculating the difference between dates and times
The DATEDIFF function can calculate the difference between two dates or datetimes in a number of units. It is similar to the DATEADD function in that you need to specify a unit for the timeperiod. The syntax is DATEDIFF('unit of timeperiod, e.g. day or hour','start date','end date').
```SQL
SELECT *
	,DATEDIFF(year,dob,SYSDATETIME()) as agenow
FROM person.info;

SELECT *
	,DATEDIFF(year,t2.dob,t1.vDate) as ageAtVisit
FROM person.schedule AS t1
LEFT JOIN person.info AS t2
	on t1.subjectID=t2.subjectID;
```
### Combining into date or extracting from date
Dates can be created by providing integers for the year, month, and day using the DATEFROMPARTS(year,month,day) syntax. This can be useful when dates in the original dataset are recorded in an awkward format and need to be imported as a text string. 
```SQL
--Example of awkward dates
CREATE TABLE #tempdate
(weirddate varchar(10) NULL); 
GO
INSERT INTO #tempdate (weirddate)
VALUES ('12/31/2019'),('5/6/19');

SELECT*
FROM #tempdate;

ALTER TABLE #tempdate
ADD vYear int
	,vMonth int
	,vDay int
	,vDate date;

UPDATE #tempdate
	SET vMonth = CAST(LEFT(weirddate,CHARINDEX('/',weirddate)-1) AS int);
		
UPDATE #tempdate
	SET vDay = CAST(SUBSTRING(weirddate,LEN(vMonth)+2,CHARINDEX('/',weirddate,LEN(vMonth)+1)-1) AS int);

UPDATE #tempdate
	SET vYear = CAST(RIGHT(weirddate,LEN(weirddate)-(LEN(vMonth)+LEN(vDay)+2)) AS int);

UPDATE #tempdate
	SET vDate = DATEFROMPARTS(vYear,vMonth,vDay);
 
UPDATE #tempdate
	SET vDate = DATEADD(year,2000,vDate)
WHERE vDate < '2000-01-01';
```
The DATEPART function can be used on existing dates. It can extract any part of a date, but is particularly useful to extract the weekday or the day of the year. 
```SQL
SELECT*
	,DATEPART(weekday,dob) as dobWeekday --in the default setting Sunday = 1!
	,DATEPART(quarter,dob) as dobQuarter
FROM person.info;
```

## Logical expressions
The CHOOSE function can be used to select something from a list based on an integer value. The syntax is CHOOSE(integer index, 'list of things to choose from separated by commas').
```SQL
SELECT*
	,CHOOSE(DATEPART(weekday,dob),'Sun','Mon','Tue','Wed','Thu','Fri','Sat') as dobWeekday
FROM person.info;
```
The most common logical expressions are conditional statements (aka if...else statements). SQL Server uses the CASE syntax for conditional statements.

CASE 'name of a column to be checked or can be left blank'

WHEN 'condition to be checked'

THEN 'what to do if condition is true

ELSE 'what to do if condition is false, this is optional, if no else option is specified, cells where the condition is false will be set to NULL'

END

You can use multiple WHEN ... THEN... statements.
```SQL
ALTER TABLE person.info
ADD gender_case varchar(6);
GO
UPDATE person.info
	SET gender_case = 
		CASE sex
			WHEN 1 THEN 'female'
			ELSE 'male'
		END;

ALTER TABLE health.physical
ADD bmi_cat_number int NULL;
GO
UPDATE health.physical
	SET bmi_cat_number = 
		CASE
			WHEN bmi_cat = 'normal' THEN 0
			WHEN bmi_cat = 'overweight' THEN 1
			WHEN bmi_cat = 'underweight' THEN -1
		END;
```		
The same operators that are used in where clauses, can be used in CASE statements.
```SQL
SELECT *
	,CASE
		WHEN height>190 AND weight<60 THEN 'tall & light'
	END AS bmi_text
FROM health.physical
```
# Exercise
Add a column to the person.info table called postcode8. In this column show the postcode with 8 characters,i.e. if a postcode is already 8 characters long, copy it as is, if it is shorter add more whitespace to the whitespace in the middle, so that it becomes 8 characters long.
<details>
	<summary>Click to see solution</summary>

```SQL	
ALTER TABLE person.info
ADD postcode8 varchar(8) NULL;
GO
UPDATE person.info
	SET postcode8 =
		CASE
			WHEN LEN(postcode) = 8 THEN postcode
			WHEN LEN(postcode) = 7 THEN REPLACE(postcode,' ','  ')
			WHEN LEN(postcode) = 6 THEN REPLACE(postcode,' ','   ')
		END;

SELECT *
	,LEN(postcode8)
FROM person.info;
```

</details>


[Previous: 3. Select queries](Section3_select.md)

[Next: 5. Aggregating data](Section5_group.md)


[Table of contents](index.md)
