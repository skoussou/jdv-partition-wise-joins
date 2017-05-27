# Partition-wise Joins in JBoss Data Virtualization

For more details about **Partition-wise Joins**, read up on the article provided below the description for this repo.

## Local Setup

### Prepare the databases and create partitions

Using the instructions from [here](https://github.com/vchintal/dockerfiles/tree/master/jboss-data-virtualization-sources/mysql). Start three MySQL instances with customer data and bind them to 3307, 3308 and 3309 respectively

```sh 
docker run -d --name mysql1 -e MYSQL_ROOT_PASSWORD=dvadmin -p 3307:3306 vchintal/mysql
docker run -d --name mysql2 -e MYSQL_ROOT_PASSWORD=dvadmin -p 3308:3306 vchintal/mysql
docker run -d --name mysql3 -e MYSQL_ROOT_PASSWORD=dvadmin -p 3309:3306 vchintal/mysql
```
Using the **Database Development** perspective of **JBoss Developer Studio** establish connections to the following 3 JDBC urls with username: **root** and password: **dvadmin**
1. jdbc:mysql://localhost:3307/uscustomers (call it _MySQL 1_)
2. jdbc:mysql://localhost:3308/uscustomers (call it _MySQL 2_)
3. jdbc:mysql://localhost:3308/uscustomers (call it _MySQL 3_)

Using SQL Scapbook on **MySQL 1** run the following set of sql commands :
```sql
delete from accountholdings where AccountID in (select AccountID from account where SSN not in ('CST01002','CST01003','CST01004','CST01005','CST01006'));
delete from account where SSN not in ('CST01002','CST01003','CST01004','CST01005','CST01006');
delete from customer where SSN not in ('CST01002','CST01003','CST01004','CST01005','CST01006');
```

Using SQL Scapbook on **MySQL 2** run the following set of sql commands :
```sql
delete from accountholdings where AccountID in (select AccountID from account where SSN not in ('CST01007','CST01008','CST01009','CST01015','CST01019'));
delete from account where SSN not in ('CST01007','CST01008','CST01009','CST01015','CST01019');
delete from customer where SSN not in ('CST01007','CST01008','CST01009','CST01015','CST01019');
```

Using SQL Scapbook on **MySQL 3** run the following set of sql commands :
```sql
delete from accountholdings where AccountID in (select AccountID from account where SSN not in ('CST01020','CST01021','CST01022','CST01027','CST01034','CST01035','CST01036'));
delete from account where SSN not in ('CST01020','CST01021','CST01022','CST01027','CST01034','CST01035','CST01036');
delete from customer where SSN not in ('CST01020','CST01021','CST01022','CST01027','CST01034','CST01035','CST01036');
```
After you run all the above delete statements against respective database instances, the **customer** and **account** tables are truly partitioned across all MySQL instances. At this point, you can optionally change the username/password for all the three connection profiles to **dvuser/dvuser**.

### Import the project 

Using the **Teiid Designer** perspective in **JBoss Developer Studio** :
1. Right click anywhere in the model explorer
2. Choose **Import → Projects from Git (with smart import) → Clone URI → https://github.com/vchintal/jdv-partition-wise-joins**
3. Start the JBoss Data Virtualization runtime associated with Designer to start testing 
4. For each **MySQL_Instance_X** source model, right click → Modeling → Set Connection Profile → Database Connections → MySQL X (where X in 1..3)
5. For each **MySQL_Instance_X** source model, right click → Modeling → Set Data Source JNDI Name → MySQL_X (where X in 1..3)

## Testing

* Run _Modeling → Preview Data_ on **customer_account_map** of MySQL.xmi virtual model. Verify the **Teiid Execution Plan**
* Run _Modeling → Preview Data_ on **customer_account_map_pu** of MySQL.xmi virtual model. Verify the **Teiid Execution Plan**

The behavior should exactly mirror the conclusion of the article. 
