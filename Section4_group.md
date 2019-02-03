# 4. Aggregating data
## Connect to database
```SQL
USE ExampleDatabase;
```
SQL Server is not designed for statistical analyses; however, it does have basic aggregate functions built in that provide descriptive statistics. When aggregate functions are used in a simple SELECT query (i.e. without a group_by or window function), they will calculate a value for the entire column. All aggregate functions have an optional ALL or DISTINCT statement at the beginning. If nothing is specified, ALL will used, which means all values will be used. If DISTINCT is specified, only unique values will be used. 
```SQL
SELECT COUNT(ALL smoker) AS allValues --counts number of values
	,COUNT(DISTINCT smoker) AS uniqueValues --counts number of unique values
	,COUNT(smoker) AS defaultMethod --default is to count all values
FROM questionnaire.recruitment
```
The DISTINCT function can also be used on its own to show all unique values in a column.
```SQL
SELECT DISTINCT(smoker)
FROM questionnaire.recruitment
```
Built in aggregate functions
```SQL
SELECT COUNT(concentration) AS n
	,AVG(concentration) AS mean
	,MIN(concentration) AS minimum
	,MAX(concentration) AS maximum
	,STDEV(concentration) AS standardDeviation
	,SUM(concentration) AS sumConcentration
	,VAR(concentration) as variance
FROM env.pollutant
```
There are functions for median, first, last etc but these can only be used with a window function (will be explained in a bit).

## Grouping
The GROUP BY function is one of the great strengths of SQL Server, because it can be used to aggregate data very quickly based on one or more columns. The GROUP BY syntax comes after the FROM syntax.
```SQL
SELECT subjectID
	,rep
	,COUNT(concentration) as n
	,MIN(concentration) as minimum
	,AVG(concentration) as mean
	,MAX(concentration) as maximum
FROM env.pollutant
GROUP BY subjectID,rep
```
GROUP BY can be used with WHERE clauses and with ORDER BY clauses. The GROUP BY syntax is always written after the WHERE clause. 

```SQL
SELECT subjectID
	,rep
	,COUNT(concentration) as n
	,MIN(concentration) as minimum
	,AVG(concentration) as mean
	,MAX(concentration) as maximum
FROM env.pollutant
WHERE rep=1
GROUP BY subjectID,rep
```
But it is always written before the ORDER BY clause. 
```SQL
SELECT subjectID
	,rep
	,COUNT(concentration) as n
	,MIN(concentration) as minimum
	,AVG(concentration) as mean
	,MAX(concentration) as maximum
FROM env.pollutant
WHERE subjectID IN (13,23,33,43,53)
GROUP BY subjectID,rep  
ORDER BY rep DESC 
```

The HAVING clause can be used to add a selection criteria to the results of a GROUP BY statement. The difference between WHERE and HAVING are that WHERE will select rows before the GROUP BY function, while HAVING will select rows after the GROUP BY function. This means HAVING can only be used when GROUP BY is present and it should be written after the GROUP by syntax.

```SQL
SELECT subjectID
	,rep
	,COUNT(concentration) as n
	,MIN(concentration) as minimum
	,AVG(concentration) as mean
	,MAX(concentration) as maximum
FROM env.pollutant
GROUP BY subjectID,rep
HAVING COUNT(concentration)>3500;  
```
### Exercise
In the health.bloodpressure table find subjects that had no variation in their pulse measurements on a visit day, i.e. within a subjectID and rep group the three measurements for pulse are identical. 
<details>
	<summary>Click to see solution</summary>

```SQL
SELECT subjectID
	,rep
	,min(pulse) as minimum
	,max(pulse) as maximum
FROM health.bloodpressure
GROUP BY subjectID,rep
HAVING min(pulse)=max(pulse)
```

</details>

## Windowing functions
Windowing functions apply a function over a set (aka window) of rows. In the broadest sense this could be all rows or it could be a number of groups (aka partitions) of rows. Unlike aggregate functions and the GROUP BY function, windowing functions do not collapse the data, but add values for each row of the dataset. This means windowing functions can use ranking functions, such as row number, rank and percentile, as well as aggregate function. Windowing functions can only be run in a SELECT query and they use the OVER syntax. The syntax in brackets after OVER specifies the window. To run an aggregate functions on the whole column, this should be left blank. To run a ranking function a column name needs to be specified inside the brackets preceded by ORDER BY. The ranking function will run in ascending order, unless otherwise specified.  

```SQL
SELECT subjectID
	,height
	,AVG(height) OVER () AS mean --adds a column showing the mean of all heights, the same value will be shown in each row 
	,COUNT(height) OVER () as n --adds a column showing the total number of heights, the same value will be shown in each row
	,ROW_NUMBER() OVER (ORDER BY subjectID) AS rowNumber -- adds row numbers based on subjectID
	,ROW_NUMBER() OVER (ORDER BY subjectID DESC) AS rowNumber_desc --adds row numbers based on subjectID in descending order
	,RANK() OVER (ORDER BY height) AS heightRank --adds rank of the height
	,DENSE_RANK() OVER (ORDER BY height) AS heightDenseRank --adds ranks as continuous numbers
	,NTILE(4) OVER (ORDER BY HEIGHT) as quartile --divides data into four equal sized groups based on rank
FROM health.physical
ORDER BY heightRank
```

The above sytax runs over all rows in a column. To run functions over smaller windows of the data, the PARTITION BY function can be added. PARTITION BY is added inside the OVER clause before the ORDER BY statement.

```SQL
SELECT*
	,ROW_NUMBER() OVER (PARTITION BY subjectID ORDER BY vDateTime_) AS rowNumber
	,COUNT(vDateTime_) OVER (PARTITION BY subjectID) AS n
FROM health.bloodpressure;
```

In the above code subjectID is the window. Consecutive numbers are added for each measurement within subjectID and they are ordered by date and time. The n column shows the total number of measurements for each subject. 
```SQL
--Example with a larger dataset
SELECT*
	,ROW_NUMBER() OVER (PARTITION BY subjectID,rep ORDER BY vDateTime) as rowNumber
	,COUNT(vDateTime) OVER (PARTITION BY subjectID,rep) as n
	,AVG(concentration) OVER (PARTITION BY subjectID,rep) as meanConc
FROM env.pollutant
```

### Lead and Lag
The LEAD and LAG function can show results from a later or previous row. You can specify the number of rows to shift by and the shift can be based on the ranking in the same column or in a different column. For example, to add the concentration from one minute before and one minute after next to the current row. 
```SQL
SELECT *
	,LAG(concentration,6,NULL) OVER (PARTITION BY subjectID,rep ORDER BY vDateTime) as previousMinute
	,LEAD(concentration,6,NULL) OVER (PARTITION BY subjectID,rep ORDER BY vDateTime) as nextMinute
FROM env.pollutant
WHERE subjectID<10;
```

### Percentiles
The above code showed that the RANK and DENSE_RANK functions can be used to add the rank of each cell as an integer. The PERCENT_RANK and CUME_DIST functions can be used to calculate the rank as a percentage. PERCENT_RANK calculates the percentage of values less than the current value in the window. This excludes the highest values in the window, which will always be assigned 1. The CUME_DIST() function calculates the percentage of values less than or equal to the current value in the group.  
```SQL
SELECT*
	,ROW_NUMBER() OVER (PARTITION BY subjectID,rep ORDER BY vDateTime) as rowDateTime
	,COUNT(vDateTime) OVER (PARTITION BY subjectID,rep) as n
	,PERCENT_RANK() OVER (PARTITION BY subjectID,rep ORDER BY vDateTime) as rowPercentRank
	,CUME_DIST() OVER (PARTITION BY subjectID,rep ORDER BY vDateTime) as rowCumeDist
FROM env.pollutant 
WHERE subjectID<10
--cumulative distance for concentration
SELECT*
	,RANK() OVER (PARTITION BY subjectID,rep ORDER BY concentration) as rankConc
	,ROW_NUMBER() OVER (PARTITION BY subjectID,rep ORDER BY concentration) as rowConc
	,COUNT(concentration) OVER (PARTITION BY subjectID,rep) as n
	,CUME_DIST() OVER (PARTITION BY subjectID,rep ORDER BY concentration) as concCumeDist
FROM env.pollutant 
WHERE subjectID<10 	
ORDER BY subjectID, rep,rowConc
```

In addition to showing the percentage rank, SQL Server can extract the percentile value (e.g. the 50th percentile aka median) from a window using the PERCENTILE_CONT and PERCENTILE_DISC functions. The difference between these two is that PERCENTILE_CONT will interpolate values, while PERCENTILE_DISC will select the closest matching lower value. For example, in a dataset with 4 different values PERCENTILE_CONT will calculate the median as the mean of the second ranked and third ranked value, while PERCENTILE_DISC will calculate the median as the second ranked value. For some reason PERCENTILE_CONT and PERCENTILE_DISC use a slightly different syntax compared to the previous ranking fiunctions: PERCENTILE_CONT('percentile to be used, e.g. 0.5 for median) WITHIN GROUP(ORDER BY 'a column name') OVER(PARTITION BY 'columns to be used for windowing')   
```SQL
SELECT *
	,PERCENTILE_CONT(0.5) WITHIN GROUP(ORDER BY pulse) OVER(PARTITION BY subjectID,rep) As medianPulseDay
	,PERCENTILE_DISC(0.5) WITHIN GROUP(ORDER BY pulse) OVER(PARTITION BY subjectID,rep) As percDiscFunc
	,COUNT(pulse) OVER (PARTITION BY subjectID) AS N
	,PERCENTILE_CONT(0.5) WITHIN GROUP(ORDER BY pulse) OVER(PARTITION BY subjectID) As medianPulseAll
	,PERCENTILE_DISC(0.5) WITHIN GROUP(ORDER BY pulse) OVER(PARTITION BY subjectID) As percDiscFunc2
FROM health.bloodpressure
```

[Previous: 3. Update queries](Section3_update.md)

[Next: 5. Join](Section5_join.md)

[Table of contents](index.md)
