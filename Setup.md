# Set up 

## Before you start
Got to [https://github.com/anmolter/sql](https://github.com/anmolter/sql) and download the following files to your P drive: 
- chick_wide.csv
- chick_long.csv
- ExampleDatabase.bak

## Installing SQL Server
Students and staff can download SQL Server 2012 Express edition from the University of Manchester Software Center. Two pieces of software are required: SQL Server 2012 Express SP2 LocalDB and SQL Server Management Studio (SSMS) 17.9. SSMS is used to interact with SQL Server databases and the data in the databases. 

![](/images/software.JPG)

### Troubleshooting
When you start SQL Server Management Studio for the first time and the Server name field on the Connect to Server window is empty: 
- start command prompt
- Run the following code: 
```cmd
sqllocaldb create "Teaching" -s
```

## Creating a database
If SQL Server and SQL Server management studio have been installed successfully, start SQL Server management studio. On the Connect to Server window, click the connect button. 

![](/images/connect.JPG)

SSMS will initially look like this: 

![](/images/launch.JPG)

On the left hand side is the Object Explorer. This shows the server that SSMS is connected to. In this case this is a local server, called Teaching. Click on the + nex to Databases. Currently there are no databases on this server other than the System Databases. Typically, you should leave the System  Databases. However, it should be noted that temporary tables are stored in tempdb during a session (at the end of a session all tables in tempdb are dropped, i.e. they are deleted).

![](/images/ObjectExplorer.jpg)

In the Object Explorer right click on *Databases*, then click *New Database...* . The New Database window will open. 

![](/images/NewDB1.JPG)

In the *Database name:* field type *MyFirstDB*. In the *Owner:* field delete <default> and type in *sa*. This sets the owner to SQL internal administrator and avoids problems, when different users need to access the database.  Look at the table under *Database files:*. The *Logical Name* has changed to MyFirstDB and MyFirstDB_log. 
  
![](/images/NewDB2.JPG) 
 
 Go to the *Initial Size (MB)* column. For a small database set the value for the MyFirstDB row to 256 and set the value for the MyFirstDB_log row to 128.
 
![](/images/NewDB3.JPG) 
 
 Next go to the *Autogrowth / Maxsize* column and click on the ... button in the MyFirstDB row. 
 
![](/images/NewDB4.JPG)
 
A new window will open called Change Autogrowth for MyFirstDB. Under File Growth select In Megabytes and change the value to 256. Under Maximum File Size select Unlimited. Then click OK. 
 
![](/images/NewDB5.JPG)
 
Click on the ... button under *Autogrowth / Maxsize* in the MyFirstDB_log row. A new window will open called Change Autogrowth for MyFirstDB_log. Under File Growth select In Megabytes and change the value to 128. Under Maximum File Size select Unlimited. Click OK.

![](/images/NewDB6.JPG)

Back on the New Database window you can see that the fields under *Autogrowth / Maxsize* have changed to *By 256 MB, Unlimited* and to *By 128 MB, Unlimited*. The *Path* column shows the directory where the database will be created. Ideally, the path should be changed so that the database is created on a different drive than the Operating System and ideally the database, log and tempdb should be created on separate, dedicated drives. However, on university managed desktops this is not possible, therefore leave the path as it is.  

![](/images/NewDB7.JPG)

On the left side of the window under *Select a page* click on *Options*. Leave the *Collation:* field as <default>. Collation is about language compatibility of the database and it is best to leave this as the server default. Ensure that the *Recovery model:* fields is set to *Simple*. The simple recovery model means that the log is recycled, which will keep it at a manageable size. The full recovery model is used when very frequent backups are required, which will also involve log backups. The Bulk-logged model is only used in very specific cases. Leave the *Compatibility level:* and *Containment type:* fields as they are. 
  
Under *Automatic* check that *Auto Create Statistics* is set to TRUE and that *Auto Shrink* is set to FALSE. Never set *Auto Shrink* to TRUE, as this can corrupt the database! Leave all other field as they are. 

![](/images/NewDB8.JPG)

You could now click OK in the bottom right to create the database. Alternatively, you can click on *Script* (top bar, above *Collation:*) to automatically write the syntax for creating the database to the Query window. Using syntax will show you all of the settings for the database, therefore click *Script* and then click the Cancel button of the New Database window. 

![](/images/NewDB9.JPG)

Scroll through the code in the Query window to have a look at it. A lot of this code specifies default settings and therefore is not necessary. In fact, the minimum code necessary to create the database is: 

```SQL
CREATE DATABASE [MyFirstDB] ON PRIMARY

( NAME = N'MyFirstDB', FILENAME = N'C:\Users\monsfam2\MyFirstDB.mdf' , SIZE = 262144KB , FILEGROWTH = 262144KB )
 LOG ON 
( NAME = N'MyFirstDB_log', FILENAME = N'C:\Users\monsfam2\MyFirstDB_log.ldf' , SIZE = 131072KB , FILEGROWTH = 131072KB );
GO

ALTER DATABASE [MyFirstDB] SET RECOVERY SIMPLE; 
GO
```

