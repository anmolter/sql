# 7. Combining queries
## Connect to database
```SQL
USE ExampleDatabase;
```

The previous sections provide the building blocks that can be used to build complex queries. Two commonly used techniques to combine multiple queries are subqueries and common table expressions. 

## Subqueries 
One or more queries can be nested in the WHERE clause of a query. A common use for this is to create a list with the nested query and then to select from the main query based on the list. The section on select queries showed that to evaluate against a list the keyword IN is used.
```SQL
SELECT subjectID
	,rep
	,dayrep
	,BP_systolic
	,BP_diastolic
FROM health.bloodpressure
WHERE subjectID IN 
	(SELECT subjectID
	FROM questionnaire.recruitment
	WHERE smoker=2);
```
The same result could be achieved with an INNER JOIN:
```SQL
SELECT b.subjectID
	,b.rep
	,b.dayrep
	,b.BP_systolic
	,b.BP_diastolic
FROM questionnaire.recruitment as a
INNER JOIN health.bloodpressure AS B
	ON a.subjectID=b.subjectID
WHERE smoker=2;
```

Adding a group by statement:
```SQL
SELECT subjectID
	,dayrep
	,AVG(BP_systolic) as meanSystole
	,AVG(BP_diastolic) AS meanDiastole
FROM health.bloodpressure
WHERE subjectID IN 
	(SELECT subjectID
	FROM questionnaire.recruitment
	WHERE smoker=2)
GROUP BY subjectID,dayrep
ORDER BY subjectID,dayrep
```
Adding a join:
```SQL
SELECT a.subjectID
	,a.dayrep
	,AVG(a.BP_systolic) as meanSystole
	,AVG(a.BP_diastolic) AS meanDiastole
	,AVG(b.temperature) AS dayTemp
FROM health.bloodpressure AS a
LEFT JOIN env.centralweather AS b
	ON a.date=b.date
WHERE subjectID IN 
	(SELECT subjectID
	FROM questionnaire.recruitment
	WHERE smoker=2)
GROUP BY subjectID,dayrep
ORDER BY subjectID,dayrep
```
To exclude data based on a subquery the NOT IN keyword can be used.
```SQL
SELECT *
FROM env.centralweather
WHERE date NOT IN (
	SELECT CONVERT(date,vDateTime)
	FROM env.temperature)
ORDER BY date;
```
## Common table expressions
Common table expressions create a temporary query result that can be used by another query. To create a common table expression the WITH 'temporary name for query' AS ('select query') syntax is used. 

The following code calculates the mean concentration per subject and repeat in the env.pollutant table and then joins the results to the questionnaires.visit table.
```SQL
WITH poll AS
	(SELECT subjectID
		,rep
		,AVG(concentration) AS meanConc
	FROM env.pollutant
	GROUP BY subjectID,rep)
SELECT a.subjectID
	,a.repVisit
	,a.smoking
	,b.meanConc
FROM questionnaire.visit as a
LEFT JOIN poll as b
	ON a.subjectID=b.subjectID AND a.repVisit=b.rep
```

Joining two grouped tables:
```SQL
WITH poll AS
	(SELECT subjectID
		,rep
		,AVG(concentration) AS meanConc
	FROM env.pollutant
	GROUP BY subjectID,rep)
SELECT a.subjectID
	,a.rep
	,AVG(a.temperature) as meanTemp
	,AVG(poll.meanConc) as meanConc
FROM env.temperature as a
LEFT JOIN poll
	ON a.subjectID=poll.subjectID AND a.rep=poll.rep
GROUP BY a.subjectID,a.rep
```

Excluding outliers with windowing function:
```SQL
WITH cte AS (
	SELECT *
		,CUME_DIST() OVER (PARTITION BY subjectID,rep ORDER BY temperature) as tempCumeDist
	FROM env.temperature
	)
SELECT subjectID
	,rep
	,vDateTime
	,temperature
FROM cte
WHERE tempCumeDist BETWEEN 0.25 AND 0.75
ORDER BY subjectID,rep,vDateTime
```

Multiple common table expressions can be used. If multiple common table expressions are used, they need to be sperated by a comma. The keyword WITH is only used once at the beginning of the code.
```SQL
WITH poll AS(
	SELECT subjectID
		,rep
		,CONVERT(VARCHAR(16),vDateTime,120) as vMinute
		,AVG(concentration) as meanConc
	FROM env.pollutant
	GROUP BY subjectID,rep,CONVERT(VARCHAR(16),vDateTime,120)
	),
temp AS(
	SELECT subjectID
		,rep
		,CONVERT(VARCHAR(16),vDateTime,120) as vMinute
		,AVG(temperature) as meanTemp
	FROM env.temperature
	GROUP BY subjectID,rep,CONVERT(VARCHAR(16),vDateTime,120)
	)
SELECT poll.*
	,temp.*
FROM poll
INNER JOIN temp
	ON poll.subjectID=temp.subjectID AND poll.rep=temp.rep AND poll.vMinute=temp.vMinute
```

Changing day repeats in health.bloodpressure into wide format:
```SQL
WITH t1 AS (
	SELECT subjectID
		,rep 
		,vDateTime_ AS morningTime
		,pulse AS morningPulse
	FROM health.bloodpressure
	WHERE dayrep=0),
t2 AS (
	SELECT subjectID
		,rep 
		,vDateTime_ AS midTime
		,pulse AS midPulse
	FROM health.bloodpressure
	WHERE dayrep=1),
t3 AS (		 
	SELECT subjectID
		,rep 
		,vDateTime_ AS eveningTime
		,pulse AS eveningPulse
	FROM health.bloodpressure
	WHERE dayrep=2)
SELECT t0.subjectID
	,t0.repVisit
	,t1.morningTime
	,t1.morningPulse
	,t2.midTime
	,t2.midPulse
	,t3.eveningTime
	,t3.eveningPulse
FROM person.schedule AS t0
LEFT JOIN t1 
	ON t0.subjectID=t1.subjectID AND t0.repVisit=t1.rep
LEFT JOIN t2
	ON t0.subjectID=t2.subjectID AND t0.repVisit=t2.rep
LEFT JOIN t3
	ON t0.subjectID=t3.subjectID AND t0.repVisit=t3.rep;
```

Adding pollutant concentration during morning based on measurement times: 
```SQL
WITH t1 AS (
	SELECT subjectID
		,rep 
		,vDateTime_ AS morningTime
		,pulse AS morningPulse
	FROM health.bloodpressure
	WHERE dayrep=0),
t2 AS (
	SELECT subjectID
		,rep 
		,vDateTime_ AS midTime
		,pulse AS midPulse
	FROM health.bloodpressure
	WHERE dayrep=1),
t3 AS (		 
	SELECT subjectID
		,rep 
		,vDateTime_ AS eveningTime
		,pulse AS eveningPulse
	FROM health.bloodpressure
	WHERE dayrep=2)
,tfinal AS(
	SELECT t0.subjectID
		,t0.repVisit
		,t1.morningTime
		,t1.morningPulse
		,t2.midTime
		,t2.midPulse
		,t3.eveningTime
		,t3.eveningPulse
	FROM person.schedule AS t0
	LEFT JOIN t1 
		ON t0.subjectID=t1.subjectID AND t0.repVisit=t1.rep
	LEFT JOIN t2
		ON t0.subjectID=t2.subjectID AND t0.repVisit=t2.rep
	LEFT JOIN t3
		ON t0.subjectID=t3.subjectID AND t0.repVisit=t3.rep
	)
SELECT 
	tfinal.subjectID
	,tfinal.repVisit
	,AVG(tfinal.morningPulse) as morningPulse
	,AVG(tfinal.midPulse) AS midPulse
	,AVG(tfinal.midPulse-tfinal.morningPulse) AS changePulse
	,AVG(poll.concentration) as morningConc
FROM tfinal
LEFT JOIN env.pollutant AS poll
	ON tfinal.subjectID=poll.subjectID AND tfinal.repVisit=poll.rep
WHERE poll.vDateTime>DATEADD(minute,10,tfinal.morningTime) AND poll.vDateTime<DATEADD(minute,-10,tfinal.midTime)
GROUP BY tfinal.subjectID,tfinal.repVisit
```

[Previous: 6. Joining data](Section6_join.md)


[Table of contents](index.md)
