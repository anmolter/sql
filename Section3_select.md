# 3. Select Queries
## Connect to database
First of all, tell SQL Server to use the ExampleDatabase.
```SQL
USE ExampleDatabase;
```
## Basic select query
Select queries are used to get data from a table. The most basic form is:
```SQL
SELECT *
FROM person.info;
```
This query gets all columns and all rows from the person.info table and displays them in the Results window. This query is very useful to quickly view data stored in a table; however, if the table is very large, it may take a while to get all of the data. In that case it may be better to restrict the select query to to the top n rows, by using:
```SQL
SELECT TOP(10)*
FROM person.info;
```
This query only shows the first 10 rows of the person.info table (it is similar to the head function in R). 

The SELECT part of the query tells SQL which columns to get from the table. If there is a * (aka wildcard) after select, then all columns will be selected. However, it is possible to select only a subset of columns, by separating the column names with commas:
```SQL
SELECT dob
	,sex
FROM person.info;
```
## Aliases 
Aliases can be used to give a temporary name to a column. This means that the Results window will display the Alias name of the column, not the actual stored name of the column. To create an alias type in AS after the original column name, followed by the alias name:
```SQL
SELECT subjectID
	,dob AS DateOfBirth
FROM person.info;		
```
Typing in AS is actually not necessary for this. Many SQL users will simply type the alias name after the original name:
```SQL
SELECT subjectID
	,dob DateOfBirth
FROM person.info;
```
However, for readibility it is often better to include AS. Aliases can also be used to assign temporary names to tables, which is particularly useful in join queries, which will be explained in a lter section.
## Where clauses
The SELECT part of a query determines, which columns to get. To get a subset of rows a WHERE clause is added after the name of the table. For example, to select all current smokers from the questionnaire.recruitment table:
```SQL
SELECT *
FROM questionnaire.recruitment
WHERE smoker=2;
```
To select a subset of rows based on a text column or a date/time column, single quotes must be placed around the value:
```SQL
SELECT*
FROM questionnaire.recruitment
WHERE fav_col='yellow';

SELECT*
FROM person.info
WHERE dob='1986-05-04';
```
The WHERE clause allows the following operators: make a table with examples
<table>
	<tr>
		<th>Operator</th>
		<th>Meaning</th>
		<th>Example</th>
	</tr>
	<tr>
		<td>
			>
		</td>
		<td>
			Greater than
		</td>
		<td>
			<pre lang="sql">
			SELECT *
			FROM person.info
			WHERE dob > '1986-05-04';
			</pre>
		</td>
	</tr>
	<tr>
		<td>
			<
		</td>
		<td>
			Less than
		</td>
		<td>
			<pre lang="sql">
			SELECT *
			FROM person.info
			WHERE dob < '1986-05-04';
			</pre>
		</td>
	</tr>	
	<tr>
		<td>
			=
		</td>
		<td>
			Equals
		</td>
		<td>
			<pre lang="sql">
			SELECT *
			FROM person.info
			WHERE dob = '1986-05-04';
			</pre>
		</td>
	</tr>
	<tr>
		<td>
			<=
		</td>
		<td>
			Less than or equal to
		</td>
		<td>
			<pre lang="sql">
			SELECT *
			FROM person.info
			WHERE dob <= '1986-05-04';
			</pre>
		</td>
	</tr>
	<tr>
		<td>
			>=
		</td>
		<td>
			Greater than or equal to
		</td>
		<td>
			<pre lang="sql">
			SELECT *
			FROM person.info
			WHERE dob >= '1986-05-04';
			</pre>
		</td>
	</tr>
	<tr>
		<td>
			!=
		</td>
		<td>
			Not equal to
		</td>
		<td>
			<pre lang="sql">
			SELECT *
			FROM env.temperature
			WHERE temperature != 5
			</pre>
		</td>
	</tr>
	<tr>
		<td>
			<>
		</td>
		<td>
			Not equal to
		</td>
		<td>
			<pre lang="sql">
			SELECT *
			FROM env.temperature
			WHERE temperature <> 5
			</pre>
		</td>
	</tr>
	<tr>
		<td>
			!<
		</td>
		<td>
			Not less than
		</td>
		<td>
			<pre lang="sql">
			SELECT *
			FROM env.temperature
			WHERE temperature !< 5
			</pre>
		</td>
	</tr>
	<tr>
		<td>
			!>
		</td>
		<td>
			Not greater than
		</td>
		<td>
			<pre lang="sql">
			SELECT *
			FROM env.temperature
			WHERE temperature !> 5
			</pre>
		</td>
	</tr>	
	<tr>
		<td>
			BETWEEN
		</td>
		<td>
			Greater than lower value and less than upper value
		</td>
		<td>
			<pre lang="sql">
			SELECT *
			FROM person.info
			WHERE dob BETWEEN '1986-01-01' AND '1988-01-01';
			</pre>
		</td>
	</tr>	
	<tr>
		<td>
			NOT BETWEEN
		</td>
		<td>
			Less than lower value or greater than upper value
		</td>
		<td>
			<pre lang="sql">
			SELECT *
			FROM person.info
			WHERE dob NOT BETWEEN '1986-01-01' AND '1988-01-01';
			</pre>
		</td>
	</tr>	
	<tr>
		<td>
			LIKE
		</td>
		<td>
			Pattern occurs in string
			% wildcard for any number of characters of any type
			_ wildcard for one character of any type
		</td>
		<td>
			<pre lang="sql">
			SELECT *
			FROM questionnaire.recruitment
			WHERE fav_col LIKE 'gr%';
			SELECT *
			FROM questionnaire.recruitment
			WHERE fav_col LIKE 'gr__';
			SELECT *
			FROM questionnaire.recruitment
			WHERE fav_col LIKE '%u%';
			SELECT *
			FROM questionnaire.recruitment
			WHERE fav_col LIKE '_e%';
			</pre>
		</td>
	</tr>		
	<tr>
		<td>
			IS NULL
		</td>
		<td>
			Is missing
		</td>
		<td>
			<pre lang="sql">
			SELECT *
			FROM questionnaire.recruitment
			WHERE alcohol IS NULL;
			</pre>
		</td>
	</tr>	
	<tr>
		<td>
			IS NOT NULL
		</td>
		<td>
			Is not missing
		</td>
		<td>
			<pre lang="sql">
			SELECT *
			FROM questionnaire.recruitment
			WHERE alcohol IS NOT NULL;
			</pre>
		</td>
	</tr>	
	<tr>
		<td>
			IN
		</td>
		<td>
			In list of values
		</td>
		<td>
			<pre lang="sql">
			SELECT *
			FROM questionnaire.recruitment
			WHERE fav_col IN ('yellow','red','blue')
			</pre>
		</td>
	</tr>			
</table>

A WHERE clause can contain multiple conditions by using the AND or OR logical operators.
```SQL
SELECT *
FROM questionnaire.recruitment
WHERE alcohol = 1 AND smoker = 2;
```
If the AND operator is used a row must fulfill both conditions to be selected. If the OR operator is used all rows fulfilling either of the two conditions will be selected. This means, if OR is used in the query above more rows will be selected than if AND is used.
```SQL
SELECT *
FROM questionnaire.recruitment
WHERE alcohol = 1 OR smoker = 2;
```
The AND and OR operators can also be nested, by using brackets. The code within the brackets will be executed first.  
```SQL
SELECT *
FROM questionnaire.recruitment
WHERE (alcohol = 1 OR smoker = 2) AND fav_col LIKE 'bl%';
```
### Exercises
1. From the env.centralweather table select all dates with temperatures below zero

<details>
	<summary>Click to see solution</summary>
	
```SQL
SELECT date
FROM env.centralweather
WHERE temperature<0;
```

</details>


2. From the health.physical table select all participants taller than 195cm and smaller than 155cm.

<details>
	<summary>Click to see solution</summary>
	
```SQL
SELECT *
FROM health.physical
WHERE height < 155 OR height > 195;
--Alternative solution:
SELECT *
FROM health.physical
WHERE height NOT BETWEEN 155 AND 195;
```

</details>	


## Sorting
Rows can be sorted using ORDER BY followed by a column name. The default is to sort in ascending order. To sort in descending order DESC needs to be added after the column name.
```SQL
SELECT*
FROM person.schedule
ORDER BY vDate; --starts with oldest date

SELECT*
FROM person.schedule
ORDER BY vDate DESC; --starts with newest date
```
Rows can be sorted based on multiple columns. The sort will operate in the order that the columns appear:
```SQL
SELECT*
FROM person.schedule
ORDER BY vDate,SubjectID;--sorts by date then by subjectID

SELECT*
FROM person.schedule
ORDER BY SubjectID,vDate;--sorts by SubjectID then by date
```
If a WHERE clause and an ORDER statement are required, the WHERE clause needs to be written first.
```SQL
SELECT*
FROM person.schedule 
WHERE attend=0
ORDER BY vDate;
```

## SELECT INTO
SELECT queries provide a temporary collection of data, i.e. they do not change the tables stored in the database and they do not create new tables. If you want to store the results of SELECT query as a new table, you can use the SELECT * INTO syntax:
```SQL
SELECT *
INTO dbo.personinfo_copy
FROM person.info;  
```
In the Object Explorer right-click on Tables and click Refresh. The new table should appear. Click the + next to dbo.personinfo_copy, then click the + next to Columns. As you can see the columns from person.info have been selected into this table; however, there is no primary key or foreign key. This means that there is no clustered index and no relationship to other tables. You could create a primary key and/or foreign key using the ALTER TABLE syntax shown in the previous section. However, many SQL professionals will advise against using SELECT INTO and recommend using subqueries or common table expressions *(these will be explained later)* instead, because they do not take up disc space. An alternative to creating a permanent table with SELECT INTO is to create a temporary table. A temporay table is created by typing # before the new table name. Temporary tables are stored in tempdb (see System Databases folder) and only exist until the query window is closed. While the query window is open, temporary tables can be used like normal database tables. 
```SQL
SELECT *
INTO #missed_appointments
FROM person.schedule
WHERE attend=0;  

SELECT*
FROM #missed_appointments;  
```

[Previous: 2. Creating tables](Section2_create.md) 

[Next: 4. Update queries](Section4_update.md) 


[Table of contents](index.md)
