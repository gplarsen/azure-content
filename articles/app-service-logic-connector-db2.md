<properties
   pageTitle="Using the Microsoft API App Connector for DB2 in a Microsoft Azure App Service Logic App"
   description="How to use the DB2 Connector with Logic App triggers and actions"   services="app-service\logic"
   documentationCenter=".net,nodejs,java"
   authors="gplarsen"
   manager=""
   editor=""/>

<tags
   ms.service="app-service-logic"
   ms.devlang="multiple"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="integration"
   ms.date="07/20/2015"
   ms.author="gplarsen"/>


# Microsoft DB2 Connector #
Microsoft Azure App Service enables you to efficiently build web and mobile solutions that use a Logic app workflow to integrate applications and an API app to connect line-of-business systems. Using the Logic app graphical designer, you can define a flow that connects to a DB2 database to accomplish these common database operations:

- Process a SQL SELECT statement to read rows in a table. 

- Process a SQL INSERT statement to add rows to a table.

- Process a SQL UPDATE statement to alter rows in a table.

- Process a SQL DELETE statement to remove rows from a table. 

- Process a SQL SELECT CURSOR statement in conjunction with a SQL UPDATE WHERE CURRENT OF CURSOR statement to alter rows in a table. 

- Process a SQL SELECT CURSOR statement in conjunction with a SQL DELETE WHERE CURRENT OF CURSOR statement to remove rows from a table. 


## Logic Apps ##
Logic apps enable you to develop and deliver powerful integration solutions that automate business process, using workflow that consists of triggers and actions. *Triggers* start a new instance of a workflow based on a specific event. *Actions* are the workflow steps that follow the trigger. 

DB2 Connector supports these Logic app triggers and actions:

Trigger | Action
------- | ------
Read and edit table rows using SELECT, SELECT for UPDATE, and SELECT for DELETE | Read, write, edit table rows using SELECT, INSERT, UPDATE, DELETE, SELECT for UPDATE, SELECT for DELETE and other statements

### Triggers ###
A trigger can process a SQL SELECT, SELECT for UPDATE, or SELECT for DELETE statement to read, edit or remove one or more table rows. 

- Process a SQL SELECT COUNT statement to check for the existence of rows in a table. For example, a trigger can check for new customer order records that have not shipped (SELECT COUNT(*) FROM NEWORDERS WHERE SHIPID IS NULL).

- Process a SQL SELECT statement to read rows in a table. For example, a trigger can read new customer order records that have not shipped (SELECT * FROM NEWORDERS WHERE SHIPDATE IS NULL).

- Process a SQL SELECT FOR UPDATE statement to read rows in a table in conjunction with an UPDATE WHERE CURRENT OF cursor statement to alter rows in a table. For example, a trigger can read new customer order records that have not shipped (SELECT * FROM NEWORDERS WHERE SHIPDATE IS NULL FOR UPDATE) and update the shipping information (UPDATE NEWORDERS SET SHIPDATE = CURRENT DATE WHERE CURRENT OF &lt;CURSOR&gt;). 

- Process a SQL SELECT FOR UPDATE statement to read rows in a table in conjunction with a DELETE WHERE CURRENT OF cursor statement to remove rows in a table. For example, a trigger can read new customer order records that have not shipped (SELECT * FROM NEWORDERS WHERE SHIPDATE IS NULL FOR UPDATE) and update the shipping information (DELETE FROM NEWORDERS WHERE CURRENT OF &lt;CURSOR&gt;)). 

### Actions ###
An action can process a SQL SELECT, SELECT for UPDATE, or SELECT, INSERT, UPDATE, DELETE or other statement, read, write, edit or remove one or more table rows.

- Process a SQL SELECT statement to read rows in a table. For example, you can read one or more new order records (SELECT * FROM NEWORDERS WHERE ORDDATE = CURRENT DATE).

- Process a SQL SELECT statement to read selected columns and rows in a table. For example, you can read one or more existing order records (SELECT ORDID, SHIPDATE, SHIPID FROM NEWORDERS WHERE SHIPDATE IS NULL).

- Process a SQL INSERT statement to add rows to a table. For example, you can add one or more new customer order records (INSERT INTO NEWORDERS VALUES (99999, 1, 99.99, 10019, CURRENT DATE, CURRENT DATE + 5 DAYS, null, 10001, 0.00, 'Ernst Handel', 'Kirchgasse 6', 'Graz', 'International', '8010', 'Austria')).

- Process a SQL UPDATE statement to alter rows in a table. For example, you can change one or more existing customer order records (UPDATE NEWORDERS SET SHIPDATE = CURRENT DATE WHERE CUSTID = 99999).

- Process a SQL DELETE statement to remove rows from a table. For example, you can remove one or more existing customer records (DELETE FROM NEWORDERS WHERE ORDID = 99999).


## Create DB2 Connector API App ##
You can create a new instance of a DB2 Connector API app from within the Azure Marketplace. 

1. In the hub menu of the Azure start board, click **HOME**, and then click **Marketplace**. In the **Everything** blade, type **db2** in the **Search Everything** box, and then click **Enter**. In the search everything results pane, click **DB2 Connector**. In the **DB2 Connector** marketplace blade, click **Create**.
2. In the **DB2 Connector New API app** blade, type a **Name**, for example **Db2ConnectorNewOrders**, and then click **Package settings**.
3. In the **Package Settings** blade, define the connector app package settings. 
4. Type a **Database Connection String**, for example, **Network Address=123.34.45.57;Network Port=446;Initial Catalog=rdbnam;User ID=username;Password=password;Package Collection=pkgcol;Default Schema=defsch;Default Qualifier=defqual;**. 
4. Type a list of **Tables**, for example, **NEWORDERS**.
5. Type a list of **Procedures**, for example, **SPORDERID**.
6. Optionally, click **true** to define an **OnPremise** connection for use with the Azure Service Proxy Relay, and then type a **Service Bus Connection String**. 
7. Type a **Data Available Query** for use with a Logic App trigger, for example, type **SELECT COUNT(\*) FROM NEWORDERS WHERE SHIPDATE IS NULL**.
8. Type a **Poll Data Query** for use with a Logic App trigger, for example, type **SELECT \* FROM NEWORDERS WHERE SHIPDATE IS NULL FOR UPDATE**.
9. Type a **Poll Alter Statement** for use with a Logic App trigger, for example, type **UPDATE NEWORDERS SET SHIPDATE = CURRENT DATE WHERE CURRENT OF &lt;CURSOR&gt;**.
10. Click **OK**. Select or define values for the other settings (e.g. service plan, resource group). The settings should look as follows. Click **Create**. <br/>
![][1]

**Note:** DB2 Connector trigger is a composite operation, comprised of three statements. If you omit the Poll Alter Statement, then DB2 Connector will process the Data Available Query and Poll Data Query, to check for data and then return the data. If you omit the Poll Data Query and Poll Alter Statement, the DB2 Connector will process the Data Available Query, to check for data and then return the row count.  

### API app package settings ###

#### Database Connection String ####
String value for required semi-colon separated list of name value pairs.

Name | Required |  Description
--- | --- | ---
Network Address | Yes | String value for the required TCP/IP address or alias of the DB2 server in either IPv4 or IPv6 format.
Network Port | Yes | Integer value for the required TCP/IP port on which the DB2 database server listens for in-bound DRDA client connection requests. 
Initial Catalog | Yes | String value for the required DB2 database instance name. 
User ID | Yes | String value for the required DB2 user name identifier. 
Password | No | String value for the optional DB2 user name password. 
Package Collection | Yes | String value for the required DB2 schema collection name to instruct Connector where to define required static SQL packages, which contain DECLARE CURSOR statements used to fetch resultset rows.
Default Schema | No | String value for optional DB2 schema collection name to instruct Connector where to locate database objects when processing metadata.
Default Qualifier | No | String value for optional DB2 schema collection name to instruct DB2 database where to locate database objects when processing SQL statements. 

#### Tables ####
String value for required comma-separated list of database objects (tables, views, aliases) to enable Connector to support OData operations, generate swagger documentation and examples. 

#### OnPremise ####
Boolean value for optional on-premises deployment using Microsoft Azure Service Bus Hybrid Relay. 

#### Service Bus Connection String ####
String value for optional semi-colon separated list of name value pairs.

#### Data Available Query ####
String value for optional SQL SELECT statement for use by Logic app trigger to check for available data in a single table. For example, type SELECT COUNT(*) FROM NEWORDERS WHERE SHIPDATE IS NULL.

#### Poll Data Query ####
String value for SQL SELECT CURSOR statement for use by Logic app trigger to query data in a single table with optional FOR UPDATE clause. For example, type SELECT * FROM NEWORDERS WHERE SHIPDATE IS NULL FOR UPDATE.  

#### Poll Alter Statement ####
String value for SQL UPDATE or DELETE statement for use by Logic app trigger to alter data in a single table.
For example, type UPDATE NEWORDERS SET SHIPDATE = CURRENT DATE WHERE CURRENT OF &lt;CURSOR&gt;.

## Create Logic App using DB2 Connector as Action to Write Data ##
You can create a new Logic app from within the Azure Marketplace, and then use the DB2 Connector as an action to add new customer orders. For example, you can use the DB2 Connector **Bulk** Insert operation to process a SQL INSERT statement (INSERT INTO NEWORDERS (ORDID,ITEMS,AMOUNT,CUSTID,ORDDATE,REQDATE,SHIPDATE,SHIPID,FREIGHT,SHIPNAME,SHIPADDR,SHIPCITY,SHIPREG,SHIPZIP,SHIPCTRY) VALUES (?,?,?,?,?,?,?,?,?,?,?,?,?,?,?)).

1. In the hub menu of the Azure start board, click **NEW**, and then click **Web + Mobile**, and then click **Logic App**. 
2. In the **Create logic app** blade, type a **Name**, for example **NewOrdersDb2**.
3. Select or define values for the other settings (e.g. service plan, resource group).
4. The settings should look as follows. Click **Create**. <p/><br/>![][2]
5. In the **Triggers and actions** blade, in the **Logic App Templates** list, click **Create from Scratch**.
6. In the **Triggers and actions** blade, in the **API Apps** panel, within the resource group, click **Recurrence**.
7. On the Logic app design surface, click the **Recurrence** item, set a **Frequency** and **Interval**, for example **Days** and **1**, and then click the **checkmark** to save the recurrence item settings.
8. In the **Triggers and actions** blade, in the **API Apps** panel, within the resource group, click **DB2 Connector**.
9. On the Logic app design surface, click the **DB2 Connector** action item, click the ellipses (**...**) to expand the operations list, and then click **Insert into NEWORDERS**.
10. On the **DB2 Connector** action item, click the ellipses (**...**), and then scroll to type values for all INSERT command column parameters. For example, type these values (CUSTID: 10042; SHIPNAME: Lazy K Kountry Store; SHIPADDR: 12 Orchestra Terrace; SHIPCITY: Walla Walla; SHIPREG: WA; SHIPZIP: 99362).  
11. Click the **checkmark** to save the action settings, and then click **Save**. The settings should look as follows. <p/><br/>![][3]
12. Click to close the **Triggers and actions** blade, and then click to close the **Settings** blade.
13. In the **All runs** list under **Operations**, click the first-listed item (most recent run).
14. In the **Logic app run** blade, click the **ACTION** item.
15. In the **Logic app action** blade, click the **OUTPUTS LINK**. The outputs should look as follows. <p/><br/>![][4]
  
**Note:** DB2 Connector defines EntitySet members with attributes, including whether the member corresponds to a DB2 column with a default or generated columns (e.g. identity). Logic app displays a red asterisk next to the EntitySet member ID name, to denote DB2 columns that require values. You should not enter a value for the ORDID member, which corresponds to DB2 identity column. You may enter values for other optional members (ITEMS, ORDDATE, REQDATE, SHIPID, FREIGHT, SHIPCTRY), which correspond to DB2 columns with default values.


## Create Logic App using DB2 Connector as Action to Write Bulk Data ##
You can create a new Logic app from within the Azure Marketplace, and then use the DB2 Connector as an action to add new customer orders. For example, you can use the DB2 Connector Bulk Insert operation to process a SQL INSERT statement (INSERT INTO NEWORDERS (ORDID,ITEMS,AMOUNT,CUSTID,ORDDATE,REQDATE,SHIPDATE,SHIPID,FREIGHT,SHIPNAME,SHIPADDR,SHIPCITY,SHIPREG,SHIPZIP,SHIPCTRY) VALUES (?,?,?,?,?,?,?,?,?,?,?,?,?,?,?)).

1. In the hub menu of the Azure start board, click **NEW**, and then click **Web + Mobile**, and then click **Logic App**. 
2. In the **Create logic app** blade, type a **Name**, for example **NewOrdersBulkDb2**.
3. Select or define values for the other settings (e.g. service plan, resource group).
4. The settings should look as follows. Click **Create**. <p/><br/>![][5]
5. In the **Triggers and actions** blade, in the **Logic App Templates** list, click **Create from Scratch**.
6. In the **Triggers and actions** blade, in the **API Apps** panel, within the resource group, click **Recurrence**.
7. On the Logic app design surface, click the **Recurrence** item, set a **Frequency** and **Interval**, for example **Days** and **1**, and then click the **checkmark** to save the recurrence item settings.
8. In the **Triggers and actions** blade, in the **API Apps** panel, within the resource group, click **DB2 Connector**.
9. On the Logic app design surface, click the **DB2 Connector** action item, click the ellipses (**...**) to expand the operations list, and then click **Bulk Insert into NEWORDERS**.
10. On the **DB2 Connector** action item, type the **rows** value as an array. For example, type the following array value.

    [{"ITEMS":2,"AMOUNT":46.00,"CUSTID":10081,"SHIPNAME":"Trail's Head Gourmet Provisioners","SHIPADDR":"722 DaVinci Blvd.","SHIPCITY":"Kirkland","SHIPREG":"WA","SHIPZIP":"98034"},{"ITEMS":3,"AMOUNT":46.00,"CUSTID":10088,"SHIPNAME":"White Clover Markets","SHIPADDR":"305 14th Ave. S. Suite 3B","SHIPCITY":"Seattle","SHIPREG":"WA","SHIPZIP":"98128","SHIPCTRY":"USA"}]

    [{"ITEMS":2,"AMOUNT":46.00,"CUSTID":10081,"REQDATE":"today","SHIPNAME":"Trail's Head Gourmet Provisioners","SHIPADDR":"722 DaVinci Blvd.","SHIPCITY":"Kirkland","SHIPREG":"WA","SHIPZIP":"98034"},{"ITEMS":3,"AMOUNT":46.00,"CUSTID":10088,"REQDATE":"tomorrow","SHIPNAME":"White Clover Markets","SHIPADDR":"305 14th Ave. S. Suite 3B","SHIPCITY":"Seattle","SHIPREG":"WA","SHIPZIP":"98128","SHIPCTRY":"USA"}]

**Note:** By omitting identity columns (e.g. ORDID), nullable columns (e.g. SHIPDATE), and columns with default values (e.g. ORDDATE, REQDATE, SHIPID, FREIGHT, SHIPCTRY), DB2 database will generate values.

**Note:** By specifying "today" and "tomorrow", DB2 Connector will generate "CURRENT DATE" and "CURRENT DATE + 1 DAY" functions (e.g. REQDATE). 

11. Click the **checkmark** to save the action settings, and then click **Save**. The settings should look as follows. <p/><br/>![][6]
12. Click to close the **Triggers and actions** blade, and then click to close the **Settings** blade.
13. In the **All runs** list under **Operations**, click the first-listed item (most recent run).
14. In the **Logic app run** blade, click the **ACTION** item.
15. In the **Logic app action** blade, click the **INPUTS LINK**. The outputs should look as follows. <p/><br/>![][7]
16. In the **Logic app action** blade, click the **OUTPUTS LINK**. The outputs should look as follows. <p/><br/>![][8]


## Create Logic App using DB2 Connector as Trigger to Alter Data and Action to Read Data##
You can create a new Logic app from within the Azure Marketplace, and then use the DB2 Connector as a trigger to alter customer orders. For example, you can use the DB2 Connector Poll Data operation to process a SQL SELECT FOR UPDATE in conjunction with a SQL UPDATE statement (UPDATE NEWORDERS SET SHIPDATE = CURRENT DATE WHERE CURRENT OF &lt;CURSOR&gt;). 

Following the trigger, you can use the DB2 Connector as action to read customer orders. For example, you can use the DB2 Connector Select operation to process a SQL SELECT statement (SELECT * FROM NEWORDERS WHERE ORDID >= 90000). 

1. In the hub menu of the Azure start board, click **NEW**, and then click **Web + Mobile**, and then click **Logic App**. 
2. In the **Create logic app** blade, type a **Name**, for example **ShipOrdersDb2**.
3. Select or define values for the other settings (e.g. service plan, resource group).
4. The settings should look as follows. Click **Create**. <p/><br/>![][9]
5. In the **Settings** blade, click **Triggers and actions**.
6. In the **Triggers and actions** blade, in the **Logic App Templates** list, click **Create from Scratch**.
7. In the **Triggers and actions** blade, in the **API Apps** panel, within the resource group, click **DB2 Connector**.
8. On the Logic app design surface, click the **DB2 Connector** trigger item, click **Poll Data**, set a **Frequency** and **Interval**, for example **Days** and **1**, and then click the **checkmark** to save the trigger item settings. 
9. In the **Triggers and actions** blade, in the **API Apps** panel, within the resource group, click **DB2 Connector**.
10. On the Logic app design surface, click the **DB2 Connector** action item, click the ellipses (**...**) to expand the operations list, and then click **Select from NEWORDERS**.
11. On the **DB2 Connector** action item, type **ORDID ge 90000** for an **expression that identifies a subset of entries**.
12. Click the **checkmark** to save the action settings, and then click **Save**. The settings should look as follows. <p/><br/>![][10]
13. Click to close the **Triggers and actions** blade, and then click to close the **Settings** blade.
14. In the **All runs** list under **Operations**, click the first-listed item (most recent run).
15. In the **Logic app run** blade, click the **ACTION** item.
16. In the **Logic app action** blade, click the **OUTPUTS LINK**. The outputs should look as follows. <p/><br/>![][11]


## Create Logic App using DB2 Connector to Remove Data ##
You can create a new Logic app from within the Azure Marketplace, and then use the DB2 Connector as an action to remove customer orders. For example, you can use the DB2 Connector conditional Delete operation to process a SQL DELETE statement (DELETE FROM NEWORDERS WHERE ORDID >= 90000).

1. In the hub menu of the Azure start board, click **NEW**, and then click **Web + Mobile**, and then click **Logic App**. 
2. In the **Create logic app** blade, type a **Name**, for example **RemoveOrdersDb2**.
3. Select or define values for the other settings (e.g. service plan, resource group).
4. The settings should look as follows. Click **Create**. <p/><br/>![][12]
5. In the **Triggers and actions** blade, in the **Logic App Templates** list, click **Create from Scratch**.
6. In the **Triggers and actions** blade, in the **API Apps** panel, within the resource group, click **Recurrence**.
7. On the Logic app design surface, click the **Recurrence** item, set a **Frequency** and **Interval**, for example **Days** and **1**, and then click the **checkmark** to save the recurrence item settings.
8. In the **Triggers and actions** blade, in the **API Apps** panel, within the resource group, click **DB2 Connector**.
9. On the Logic app design surface, click the **DB2 Connector** action item, click the ellipses (**...**) to expand the operations list, and then click **Conditional delete from NEWORDERS**.
10. On the **DB2 Connector** action item, type **ORDID ge 90000** for an **expression that identifies a subset of entries**.
11. Click the **checkmark** to save the action settings, and then click **Save**. The settings should look as follows. <p/><br/>![][13]
12. Click to close the **Triggers and actions** blade, and then click to close the **Settings** blade.
13. In the **All runs** list under **Operations**, click the first-listed item (most recent run).
14. In the **Logic app run** blade, click the **ACTION** item.
15. In the **Logic app action** blade, click the **OUTPUTS LINK**. The outputs should look as follows. <p/><br/>![][14]


## DB2 Connector Samples ##
You can use the DB2 Connector with the 
DB2 Connector with Logic App walk-through exercises utilize the following table and procedure

### Create Table ###
You can create the sample NEWORDERS table using the following DDL statement.

	CREATE TABLE ORDERS (
		ORDID INT NOT NULL GENERATED BY DEFAULT AS IDENTITY (START WITH 10000, INCREMENT BY 1) , 
		CUSTID INT NOT NULL , 
		EMPID INT NOT NULL DEFAULT 10000 , 
		ORDDATE DATE NOT NULL DEFAULT CURRENT DATE , 
		REQDATE DATE DEFAULT CURRENT DATE , 
		SHIPDATE DATE , 
		SHIPID INT NOT NULL DEFAULT 10000, 
		FREIGHT DECIMAL (9,2) NOT NULL DEFAULT 0.00 , 
		SHIPNAME CHAR (40) NOT NULL , 
		SHIPADDR CHAR (60) NOT NULL , 
		SHIPCITY CHAR (20) NOT NULL , 
		SHIPREG CHAR (15) NOT NULL , 
		SHIPZIP CHAR (10) NOT NULL , 
		SHIPCTRY CHAR (15) NOT NULL DEFAULT 'USA' , 
		PRIMARY KEY(ORDID)
		)

	CREATE UNIQUE INDEX XORDID ON ORDERS (ORDID ASC)


### Create Procedure ###
You can create the sample NEWORDERS table using the following DDL statement.

	CREATE OR REPLACE PROCEDURE NWIND.SPORDERID (IN ORDERID VARCHAR(128))
		DYNAMIC RESULT SETS 1
	P1: BEGIN
		DECLARE CURSOR1 CURSOR WITH RETURN FOR
			SELECT * FROM NWIND.NEWORDERS 
				WHERE ORDID = ORDERID;
		OPEN CURSOR1;
	END P1
	')

The sample table and procedure are derived from the Microsoft Host Integration Server 2013 SDK Sample Database for DB2. Follow the instructions in the \DataIntegration\Db2Client\SampleDatabases\Readme_01_Setup.txt. 

See [HIS 2013 SDK on Microsoft Download Center](http://go.microsoft.com/fwlink/?LinkId=393355). 


## More Information ##
You can obtain more information on the DB2 Connector in the following resources. 

### DB2 Connector Overview ###
See [DB2 Connector Overview](https://azure.microsoft.com/en-us/documentation/articles/app-service-db2-overview).

### DB2 Connector API Reference ###
See [DB2 Connector API Reference](https://msdn.microsoft.com/library/azure/dn948518.aspx).

### DB2 Connector Troubleshooting ###
See [DB2 Connector Troubleshooting](https://azure.microsoft.com/en-us/documentation/articles/app-service-db2-troubleshooting).

### Hybrid Configuration ###
App Service supports a Hybrid Configuration to connect securely to your on-premises servers. 



See [Using the Hybrid Connection Manager](https://azure.microsoft.com/en-us/documentation/articles/app-service-logic-hybrid-connection-manager/).
<!--Image references-->
[1]: ./media/app-service-logic-connector-db2/ApiApp_Db2Connector_Create.jpg
[2]: ./media/app-service-logic-connector-db2/LogicApp_NewOrdersDb2_Create.jpg
[3]: ./media/app-service-logic-connector-db2/LogicApp_NewOrdersDb2_TriggersActions.jpg
[4]: ./media/app-service-logic-connector-db2/LogicApp_NewOrdersDb2_Outputs.jpg
[5]: ./media/app-service-logic-connector-db2/LogicApp_NewOrdersBulkDb2_Create.jpg
[6]: ./media/app-service-logic-connector-db2/LogicApp_NewOrdersBulkDb2_TriggersActions.jpg
[7]: ./media/app-service-logic-connector-db2/LogicApp_NewOrdersBulkDb2_Inputs.jpg
[8]: ./media/app-service-logic-connector-db2/LogicApp_NewOrdersBulkDb2_Outputs.jpg
[9]: ./media/app-service-logic-connector-db2/LogicApp_UpdateOrdersDb2_Create.jpg
[10]: ./media/app-service-logic-connector-db2/LogicApp_UpdateOrdersDb2_TriggersActions.jpg
[11]: ./media/app-service-logic-connector-db2/LogicApp_UpdateOrdersDb2_Outputs.jpg
[12]: ./media/app-service-logic-connector-db2/LogicApp_RemoveOrdersDb2_Create.jpg
[13]: ./media/app-service-logic-connector-db2/LogicApp_RemoveOrdersDb2_TriggersActions.jpg
[14]: ./media/app-service-logic-connector-db2/LogicApp_RemoveOrdersDb2_Outputs.jpg




