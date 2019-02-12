# 2. Creating Tables

### Connect to database
First we need to tell SQL which database we are working in.
```SQL
USE MyFirstDB;
```
Alternatively, type use, then click on the database name and drag and drop it next to use
```SQL
USE [MyFirstDB]
```
### Create schema
Next we create a schema. In SQL Server schemas can be used to group tables together. It is not necessary to create schemas, but they can be helpful, if there are many tables or if advanced security settings are required.  
```SQL
CREATE SCHEMA subject;
```
### Create table
We then create a table in the schema with five columns. The columns are added in brackets and separated by a comma. For each variable we need to specify the column name, the data type, if it is the primary key and if missing values are allowed. See https://www.techonthenet.com/sql_server/datatypes.php for a list of data types.
```SQL
CREATE TABLE subject.basicinfo --[schema name].[table name]
	(Subject_ID int PRIMARY KEY NOT NULL, 
	FirstName varchar(25) NOT NULL,
	Consent bit NULL,
	Height decimal(5,1) NULL,
	DOB date NULL);
 ``` 
In the Object Explorer click + next to MyFirstDB, then right click on Tables and click refresh. Click on the + next to Tables and a new table subject.basicinfo should be listed. Click on the + next to subject.basicinfo, the click on the + next to Columns. The names of the columns will appear. The data type is shown in brackets and whether missing values are allowed. PK and the grey key symbol indicate the primary key. Right click on any variable name and you will find options to modify, rename or delete the variable. If you click on modify, you can change the name and data type of each column and whether it allows missing values. The data type and missing values setting can also be changed by using syntax:
```SQL
ALTER TABLE [subject].[basicinfo]
	ALTER COLUMN DOB date NOT NULL;
```
Refresh subject.basicinfo to check that DOB has been changed to not allow missing values. By using the alter column syntax only one column can be changed at a time. Therefore, depending on the number of columns that need to be changed, it may be quicker to use the modify menu option. 

The alter table syntax can also be used to add more variables to the table. Multiple variables can be added at the same time:
```SQL
ALTER TABLE [subject].[basicinfo]
	ADD LastName varchar(40) NULL,
	Weight int NULL;
```
It can also be used to remove columns from the table. Multiple variables can be removed at the same time:
```SQL
ALTER TABLE [subject].[basicinfo]
	DROP COLUMN LastName,
	Weight; 
```
To change the name of a variable the following syntax should be used: 
```SQL
sp_rename 'table name.old column name', 'new column name', 'COLUMN';
```
For example, to change DOB to DateOfBirth:
```SQL
sp_rename '[subject].[basicinfo].[DOB]', 'DateOfBirth', 'COLUMN';
```
Refresh subject.basicinfo to see the change. Also read the warning in Messages! Renaming variables can cause problems and in practice should be avoided.

Right click on Height, then click on Properties. Under Select a page click on Extended Properties. Here you can add information that will help other users to understand the variable. Under Name type in unit, under Value type in centimetres. Click OK to save this information.     

To create a table outside of a schema, just enter a table name without the schema name.
```SQL
CREATE TABLE subjectinfo
	(Subject_ID int PRIMARY KEY NOT NULL, 
	FirstName varchar(25) NOT NULL,
	Consent bit NULL,
	Height decimal(5,1) NULL,
	DOB date NULL);  
```  
Refresh Tables in the Object Explorer. The new table will appear as dbo.subjectinfo, dbo stands for database object.

To delete a table from a database you can right click on the table and click delete or use the following syntax:
```SQL
DROP TABLE [dbo].[subjectinfo];
```
Refresh Tables to see the change. 

## Adding data 
Data can be added to a table manually. Data is added by row using the insert into syntax:
```SQL
INSERT INTO [table name] (variable1, variable2, variable3,... variableN) VALUES (value of variable1, value of variable2, value of variable3,... value of variableN);
```
For example:
```SQL
INSERT INTO [subject].[basicinfo] (
	Subject_ID
	,FirstName
	,Consent
	,Height
	,DateOfBirth
	)
VALUES	(
	1
	,'Alpha'
	,1
	,162
	,'1995-10-11'
	);
 ``` 
To quickly check the contents of the table use this code (*Select statements will be explained in the next section*):
```SQL
SELECT * 
FROM subject.basicinfo
```
To add multiple rows, you need multiple brackets after VALUES seperated by a comma:
```SQL
INSERT INTO [subject].[basicinfo] (
  Subject_ID
  ,FirstName
  ,Consent
  ,Height
  ,DateOfBirth
  )
VALUES (
  2
  ,'Beta'
  ,0
  ,187
  ,'1994-04-17'
  ),
  (
  3
  ,'Gamma'
  ,NULL
  ,171
  ,'1989-07-13'
  );
 ``` 
Use the select statement above to see how the table has changed. It is important to note that the primary key has a unique constraint, which means the same value cannot be added twice. For example the following code will result in an error message:
```SQL 
INSERT INTO [subject].[basicinfo] (
  Subject_ID
  ,FirstName
  ,Consent
  ,Height
  ,DateOfBirth
  )
VALUES (
  2
  ,'Delta'
  ,0
  ,175
  ,'1997-07-01'
  );
 ``` 
It is possible to add values to a subset of columns, by not listing all columns in the insert statement. For the columns not listed a NULL (i.e. missing) value will be added. However, if the column that is not listed does not allow missing values, an error message will be displayed. For example, this code will work:
```SQL
INSERT INTO [subject].[basicinfo] (
  Subject_ID
  ,FirstName
  ,DateOfBirth
  )
VALUES (
  4
  ,'Delta'
  ,'1989-07-01'
  );
 ``` 
But this code won't work, because DateOfBirth cannot be NULL:
``` SQL
INSERT INTO [subject].[basicinfo] (
  Subject_ID
  ,FirstName
  ,Consent
  ,Height
  )
VALUES (
  5
  ,'Epsilon'
  ,0
  ,175
  );
 ```
 
 ## Import data
In practice, INSERT INTO is only used when small manual changes are required to a table. To add large datasets to a database the import data wizard can be used. To open the wizard right click on MyFirstDB then click on Tasks > Import Flat File ... . Once the import wizard opens, click Next >. Under Location of file to be imported browse to the chick_wide.csv that you downloaded earlier. You can enter a new table name, if you wish. Under Table schema you can choose whether you want to create a table in the general database or in a schema. For this exercise select subject. Click Next >. The next page shows the first 50 rows of the data. Click Next >. The wizard will make an educated guess what the data type of each column is, but this often needs to be modified. For name change the Data Type to varchar(2); for dob change the Data Type to date, for female keep the Data Type as bit, for diet change the Data Type to tinyint. Select chickID as the Primary Key. Allow Nulls for name, dob, female and diet. Click Next >.  Click Finish. If the data is successfully imported a green tick mark will appear. Click Close. In the Object Explorer refresh Tables. The newly imported table should appear. Click the + next to columns and check that the variable specifications are correct. You can add metadata to each variable by right clicking on it, then clicking on Properties and then on Extended Properties. Run the following code to see the data:  
```SQL
SELECT * 
FROM subject.chick_wide
```
Follow the steps above to open the import wizard and browse to the chick_long.csv file. Select subject as the schema. You can see on the Preview page that these are repeated measurements on the same chicks; therefore, the chickID column alone does not uniquely identify each row. This means the primary key needs to be a compound key of multiple columns. On the Modify Columns page select chickID and time as Primary Key. Change the Data Type of weight to decimal(4,1) and tick Allow Nulls. Finish the import.  In the Object Explorer refresh Tables. The newly imported table should appear. Click the + next to columns. You can see that both chickID and time are the primary key. Click the + next to Indexes. You can see that when the primary key is created, it creates a clustered index on the primary key columns. This will speed up searches and joins based on the primary key columns. By using syntax instead of the import wizard it is possible to create a primary key that is not a clustered index. And a clustered index can then be added to columns that are not part of the primary key. It is also possible to create a table without a clustered index (and/or primary key), but this table is likely to perform badly. 

Use the following code to look at the data:
```SQL
SELECT * 
FROM subject.chick_long
```
The two tables that were just added contain data from an experiment in which newly hatched chicken were assigned to four different diets and their weight gain was monitored over a number of days. The chick_wide table contains unique data for each chicken, while the chick_long table contains repeated data for each chicken; therefore, the two tables are related via a one to many relationship. To create this relationship in the database a foreign key constraint is added to the chick_long table:  
```SQL
ALTER TABLE subject.chick_long     
ADD CONSTRAINT FK_chicklong_chickwide FOREIGN KEY (chickID) --FK_chicklong_chickwide is the name of the key     
    REFERENCES subject.chick_wide (chickID) --the table and column on the one side of the relationship     
    ON DELETE CASCADE    --to enforce referential integrity
    ON UPDATE CASCADE;  --to enforce referential integrity
 ```   
In the Object Explorer look at the columns of subject.chick_long. You can see that chickID is now a foreign key (as well as the primary key). Click on the + next to Keys. You can see that this table has two keys, a primary key and one foreign key. A table can have multiple foreign keys, but only one primary key (but the primary key can consist of multiple columns). To visualize the relationship between the tables right click on Database Diagrams then click New Database Diagram. Hold the shift key and click on chick_wide(subject) and chick_long(subject) to select them both, then click on Add and then click Close. A new tab will open showing a diagram of the two tables. It shows the name of each table with the schema name in brackets and lists the names of the columns. It also shows the primary key columns (yellow keys) and the relationship between the tables. 

The foreign key that has been set up enforces referential integrity. This means that if a record in the REFERENCES table is deleted, it will be deleted in all the related tables. For example this code will delete all data for chickID 2:
```SQL
DELETE FROM subject.chick_wide
WHERE chickID=2;
```
To check what has happened select of both of these queries and run:
```SQL
SELECT*
FROM subject.chick_wide;
SELECT*
FROM subject.chick_long;
```
All rows for chickID 2 have been deleted from both tables. Changes will cascade downwards from the REFERENCES table to the foreign key table, but not upwards. This means if a record is changed in chick_long, it will not affect chick_wide:
```SQL
DELETE FROM subject.chick_long
WHERE chickID=3;
SELECT*
FROM subject.chick_wide;
SELECT*
FROM subject.chick_long;
```
Referential integrity is very useful, for example when a study participant drops out of a study and all data from that person has to be removed. However, there is also a danger of deleting data by mistake; therefore, any delete queries should be used very carefully.

Another advantage of referential integrity is that data cannot be added to a foreign key table if the value does not exist in the REFERENCES table. For example, if you try to add data for chickID 51 to chick_long, you will get an error, because chickID 51 does not exist in chick_wide:
```SQL
INSERT INTO subject.chick_long (
  chickID
  ,time
  ,weight
  )
VALUES 
  (
  51
  ,0
  ,43.0
  );
```  
To add data for chickID 51 to chick_long, it needs to be added to chick_wide first:
```SQL
INSERT INTO subject.chick_wide (
  chickID
  ,name
  ,dob
  ,female
  ,diet
  )
VALUES (
  51
  ,'ZZ'
  ,'2019-01-01'
  ,'TRUE'
	,4);
SELECT*
FROM subject.chick_wide;
INSERT INTO subject.chick_long (
  chickID
  ,time
  ,weight
  )
VALUES (
  51
  ,0
  ,43.0
  );
SELECT*
FROM subject.chick_long;
```
## Backup database
In the Object Explorer right click on MyFirstDB, then click Tasks > Back Up... . The box under destination shows the directory, where the backup file will be saved to. If you are happy with the destination shown, click OK. To change the destination, click Add... .A new window called Select Backup Destination will open. Click the ... button and then navigate to the directory, where you would like to save the backup. Then enter a file name for your backup with a .bak extension. Click OK, click OK. The box under Destination will show two destinations, the default one and the newly added one. If you only want to save a backup to the newly added destiantion, click the default one and then click remove. Then click OK to backup the database. Depending on the size of the database, this amy take a while. 

![](/images/backup.JPG)

## Restore database from backup
In the Object Explorer right click on Databases, then click Restore Database... . Under Source select device, then click the browse button. In the new window click Add. Navigate to the location where you have saved ExampleData.bak, click on ExampleData.bak, then click OK. Click OK, then click Cancel. In the Object Explorer right click on Databases, then click Refresh. A new database called ExampleDatabase should appear. Click on the + next to ExampleDatabase, then click on the + next to DatabaseDiagrams, then doubleclick on dbo.Object_Model_Diagram. A diagram showing the relationships between all the tables should open. 

ExampleDatabase contains data from a fictious study (the "measured" values in this study are random numbers!). In this study the effect of temperature and pollution on blood pressure is measured. The study has recruited 100 participants (see person.info) and each participant has to complete 4 repeat visits (see person.schedule). During each repeat visit the participant carrys two personal monitors, which continuously measure temperature (env.temperature)and a pollutant (env.pollutant). During each repeat visit the participant's blood pressure is measured three times, i.e. in the morning, midday and in the evening (health.bloodpressure). At the end of each repeat visit the participant completes a questionnaire on smoking and exercise (questionnaire.visit) and smokers list the times when they smoked a cigarette (questionnaire.cigarettes). Before their first repeat visit, each participant has already completed a baseline questionnaire (questionnaire.recruitment) and their height and weight was measured (health.physical). In addition to these personal measurements, the researcher has also collected daily average temperatures from a Met Office monitor for a whole year (env.centralweather). Feel free to explore the columns, keys and properties of each table.    

[Previous: 1. Set up](Section1_setup.md) 

[Next: 3. Select queries](Section3_select.md) 


[Table of contents](index.md)

