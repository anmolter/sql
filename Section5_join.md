# 5. Joining data
## Connect to database
```SQL
USE ExampleDatabase;
```
## Combining data horizontally
Some software packages call this "appending data". The basic idea is that if two tables, A and B, have the same number of columns with the same data type and in the same order, then table B can be added or "appended" to the bottom of table A. This is similar to the INSERT INTO syntax, but rather than inserting data row by row, it combines whole tables. To combine tables horizontally the UNION statement is used between two SELECT queries. The default behaviour of UNION is to remove duplicate rows from the combined table, i.e. if a row in table B already exists in table A it is not appended. To append all rows from table B regardless of duplicates the UNION ALL syntax needs to be used.

Replicating a table:
```SQL
SELECT *
FROM person.info
UNION ALL
SELECT *
FROM person.info
ORDER BY subjectID;
```
Without ALL added to UNION there will be no duplicates:
```SQL
SELECT *
FROM person.info
UNION
SELECT *
FROM person.info
ORDER BY subjectID
```
Selecting rows first and then combining:
```SQL
SELECT subjectID
	,smoker
FROM questionnaire.recruitment
WHERE smoker=1
UNION
SELECT subjectID
	,smoker
FROM questionnaire.recruitment
WHERE smoker=2
```
Selecting data from different tables:
```SQL
SELECT 'personal' AS source
	,subjectID
	,CONVERT(date,vDateTime) as date
	,AVG(temperature) as temp
FROM env.temperature
GROUP BY subjectID,CONVERT(date,vDateTime)
UNION
SELECT 'central' AS source
	,NULL as subjectID 
	,date
	,temperature as temp
from env.centralweather
ORDER BY DATE;
```
SELECT INTO can be used to save the results of the UNION as a new table. However, this new table will not have primary keys or foreign keys, even if either of the original tables has primary keys or foreign keys. 
```SQL
SELECT 'personal' AS source
	,subjectID
	,CONVERT(date,vDateTime) as date
	,AVG(temperature) as temp
INTO dbo.temperatureCombined
FROM env.temperature
GROUP BY subjectID,CONVERT(date,vDateTime)
UNION
SELECT 'central' AS source
	,NULL as subjectID 
	,date
	,temperature as temp
from env.centralweather
ORDER BY DATE;

SELECT*
FROM [dbo].[temperatureCombined];
```

## Combining data vertically
SQL Server only uses the term join, when combining tables vertically. This means one or more columns from table B are added to one or more columns in table A. Table A and table B do not need to have the same number columns, type of columns or order of columns. However, they should have at least one common column that can be used as a join condition, i.e. it determines which rows correspond to each other. The syntax for joins is FROM 'name of original table that columns are joined to' JOIN 'name of new table that is providing columns' ON 'condition that idenfies which rows correspond to each other'. It is common practice to add an alias to the table names in a join query. This makes the code easier to understand and avoids problems when the same column names appear in both tables.
```SQL
SELECT tableA.* --selects all columns from tableA
	,tableB.* --selects all columns from tableB
FROM person.info as tableA
JOIN questionnaire.recruitment as tableB
ON tableA.subjectID=tableB.subjectID
```
There a several different types of joins. See Venn diagrams in powerpoint. 
### INNER JOIN
If JOIN is used without any additional keywords it defaults to being an inner join. An inner join will only keep rows that are in both tables.
```SQL
SELECT *
INTO #tableA
FROM person.info
WHERE subjectID<75;
GO
SELECT *
INTO #tableB
FROM person.info
WHERE subjectID>25;

SELECT a.*
	,b.*
FROM #tableA as a
INNER JOIN #tableB as b
	ON a.subjectID=b.subjectID

--
SELECT a.*
	,b.*
FROM person.schedule as a
INNER JOIN questionnaire.visit as b
	ON a.subjectID=b.subjectID AND a.repVisit=b.repVisit
```

### LEFT JOIN
A left join keeps everything in table A, regardless of whether it has a match in table B. It adds everything from table B that has a match in table A. If table B has multiple rows that match the same row in table A (i.e. there is a one-to-many relationship), all matching rows from table B will be added and rows from table A will be repeated.
Example of join with missing information table B:
```SQL
SELECT a.*
	,b.*
FROM person.schedule as a
LEFT JOIN questionnaire.visit as b
	ON a.subjectID=b.subjectID AND a.repVisit=b.repVisit
```
Example of join with one-to-many relationship:
```SQL
SELECT a.*
	,b.*
FROM person.schedule as a
LEFT JOIN health.bloodpressure as b
	ON a.subjectID=b.subjectID AND a.repVisit=b.rep
```
### LEFT JOIN without intersection
To call this a join is a bit misleading, because it essentially removes all rows from table A that are in table B. Nothing is added from table B and it adds NULL values for the columns in table B.
```SQL
SELECT a.*
	,b.*
FROM person.schedule as a
LEFT JOIN health.bloodpressure as b
	ON a.subjectID=b.subjectID AND a.repVisit=b.rep
WHERE b.subjectID IS NULL AND b.rep IS NULL
```
Obviously, we do not have to show the NULL columns:
```SQL
SELECT a.subjectID
	,a.vDate
	,a.attend
FROM person.schedule as a
LEFT JOIN health.bloodpressure as b
	ON a.subjectID=b.subjectID AND a.repVisit=b.rep
WHERE b.subjectID IS NULL AND b.rep IS NULL
```
### RIGHT JOIN
A right join keeps everything in table B, regardless of whether it has a match in table A. It adds everything from table A that has a match in table B. If table A has multiple rows that match the same row in table B, all matching rows from table A will be added and rows from table B will be repeated. The exact same result will be produced by using a left join and switching table A and table B, so a right join is rarely necessary.
```SQL
SELECT a.*
	,b.*
FROM person.schedule as a
RIGHT JOIN env.centralweather as b
	ON a.vDate=b.date
```
### RIGHT JOIN without intersection
This is the same as a LEFT JOIN without intersection, but in this case everything from table B is removed that is in table A. Nothing is added from table A and it adds NULL values in the columns from table A. 
```SQL
SELECT a.*
	,b.*
FROM person.schedule as a
RIGHT JOIN env.centralweather as b
	ON a.vDate=b.date
WHERE a.vDate IS NULL 
```
### FULL JOIN
Returns all rows from both tables. If the join condition has no match, i.e. a row has no correspnding row, NULL values are inserted. 
```SQL
SELECT a.*
	,b.*
FROM #tableA AS a
FULL JOIN #tableB AS b
	ON a.subjectID=b.subjectID
```
### FULL JOIN without intersection
This returns all rows from table A and all rows from table B, but removes rows that are in both tables. 
```SQL
SELECT a.*
	,b.*
FROM #tableA AS a
FULL JOIN #tableB AS b
	ON a.subjectID=b.subjectID
WHERE a.subjectID IS NULL OR b.subjectID IS NULL
```
### CROSS JOIN
A cross join produces the cartesian product of two tables. This means each row in table A is joined to each row in table B. Cross joins can result in very large tables, so they should be used very carefully.
```SQL
SELECT DISTINCT(repVisit) as rep
INTO #tableRep
FROM person.schedule;

SELECT *
FROM person.info
CROSS JOIN #tableRep
```

SQL can join multiple tables together. 
```SQL
SELECT a.subjectID
	,a.dob
	,b.bmi
	,c.smoker
FROM person.info as a
LEFT JOIN health.physical as b
	ON a.subjectID=b.subjectID
LEFT JOIN questionnaire.recruitment as c
	ON a.subjectID=c.subjectID
```
Different types of joins can be used, when joining multiple tables. 
```SQL
SELECT a.subjectID
	,a.attend
	,b.vDateTime
	,b.temperature AS persTemp
	,c.temperature AS centTemp
FROM person.schedule AS a
INNER JOIN env.temperature AS b
	ON a.subjectID=b.subjectID AND a.repVisit=b.rep
LEFT JOIN env.centralweather AS c
	ON a.vDate=c.date
```
### Exercise
Join the three tables in the questionnaire schema to person.schedule showing missing data (i.e. NULL values). From person.schedule show the subjectID, repVisit and attend column, from questionnaire.recruitment show the smoker column, from questionnaire.visit show the smoking column and from questionnaire.cigarettes show the vDateTime and cig_number column. Order by subjectId and vDateTime. 
<details>
	<summary>Click to see solution</summary>
	
```SQL	
SELECT a.subjectID
	,a.repVisit
	,a.attend
	,b.smoker
	,c.smoking
	,d.vDateTime
	,d.cig_number
FROM person.schedule as a
LEFT JOIN questionnaire.recruitment as b
	ON a.subjectID=b.subjectID
LEFT JOIN questionnaire.visit as c
	ON a.subjectID=c.subjectID AND a.repVisit=c.repVisit
LEFT JOIN questionnaire.cigarettes as d
	ON c.subjectID=d.subjectID AND c.repVisit=d.rep
ORDER BY a.subjectID,d.vDateTime
```
	
</details>


[Previous: 4. Aggregating data](Section4_group.md)

[Next: 6. Combining queries](Section6_cte.md)

[Table of contents](index.md)
