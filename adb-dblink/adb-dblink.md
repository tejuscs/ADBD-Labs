# Migrating to Autonomous AI Database Dedicated using Database Link

## Introduction

The Autonomous AI Database supports outgoing Database Links to other Oracle databases.  Database Links can be used with Autonomous AI Database to read or transfer data between 2 hosts. As long as there is right network connectivity from Source database to Autonomous AI Database, Database Links can used to Migrate the databases. 

This lab walks you through how to migrate oracle database to Autonomous AI Database using Datapump and Database Links.

Estimated Time: 40 minutes

### Objectives

As a database administrator:
1. Configure Database Links between Source and Autonomous AI Database.
2. Use Datapump to import the database from Source system to Autonomous AI Database.

### Required Artifacts

- An Oracle Cloud Infrastructure account.
- Source Database (Oracle Database Cloud Service) and Autonomous AI Database 
- Network Connectivity between Source and Autonomous AI Database

## Task 1: Test Network connection between your Source System and Autonomous AI Database

This Lab assumes you have a pre-provisioned Database Cloud service in your tenancy as a Source database. Learn how to create Database Cloud Service [here](https://docs.oracle.com/en-us/iaas/dbcs/doc/create-db-system-using-console.html).

NOTE: Network should be configured prior to next steps for Source Database to talk to Autonomous AI Database. 
Refer [here]() to setup your Network configuration. 

- Log in to your OCI tenancy and navigate to Oracle Base Database(VM,BM) from the main hamburger menu.

    ![This image shows the result of performing the above step.](./images/dblink1.png)

- Select the right Compartment and Click on your Source Database.

    ![This image shows the result of performing the above step.](./images/dblink2.png)

- SSH into your Bastion host and sqlplus to your Autonomous AI Database.

Note: Please refer to Lab guide ***Configure a development system for use with your dedicated Autonomous AI Database*** under Developer and Database Users to configure your Development system to be able to ssh or sqlplus into Oracle database. 

- Ensure the network is configured from your Source System to talk to Autonomous AI Database. Download and copy the wallet to your Source System and test the connection to ADB.

    ![This image shows the result of performing the above step.](./images/dblink3.png)



## Task 2: Create Database Link

Once you are sucessfully connected the network configuration from your Source Database to Autonomous AI Database in Task 1, we are not going to create a Database link from Autonomous AI Database to Source Oracle Database. 

- SQLPLUS into Autonomous AI Database from your Source System

    ![This image shows the result of performing the above step.](./images/dblink3.png)

- Create the the Database link from your Autonomous AI Database to Source Database 

NOTE: sqlplus into Autonomous AI Database using **admin** as Username and the Password and Create Database link.

    
![This image shows the result of performing the above step.](./images/dblink4.png)

````
<copy>
CREATE DATABASE LINK <Source_Database_global_unique_name> CONNECT TO system identified by <SourceDB_password> USING '<Connect_string_Source_DB>';
</copy>
````

- Test if the Database Link you created works from Autonomous AI Database


    ![This image shows the result of performing the above step.](./images/dblink5.png)

````
<copy>
select * from dual@<Database_Link>;
</copy>
````

NOTE: Create Profiles and Roles prior to importing 


## Task 3: Import data from Source Oracle database to Autonomous AI Database

You are now ready to import data from your Source database to Autonomous AI Database using import data pump.


- Select the schema / schemas you want to import to Autonomous AI Database and run Import datapump from your source System.


    ![This image shows the result of performing the above step.](./images/dblink6.png)

````
<copy>
impdp admin/<ADB_Password>@adb2_high SCHEMAS = <schemas> network_link=<Database_Link> parallel=1 transform=segment_attributes:n exclude=cluster nologfile=yes remap_tablespace=USERS:DATA
</copy>
````

- SQLPLUS into Autonomous AI Database to verify the data migration.

    ![This image shows the result of performing the above step.](./images/dblink7.png)

- Select values from the table you want to verify

    ![This image shows the result of performing the above step.](./images/dblink8.png)

You have now successfully migrated your Source Database to Autonomous AI Database using Database Links. 


## Acknowledgements
*Congratulations! You successfully migrated your Oracle database to Autonomous AI Databases*

- **Author** - Tejus S
- **Last Updated By/Date** -  Tejus S, July 2022

## See an issue or have feedback?  
Please submit feedback [here](https://apexapps.oracle.com/pls/apex/f?p=133:1:::::P1_FEEDBACK:1).   Select 'Autonomous DB on Dedicated Exadata' as workshop name, include Lab name and issue / feedback details. Thank you!
