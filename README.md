# jdbc-driver
Extracting of Ariel data (events, flows and simulated arcs) out of a QRadar system through a JDBC driver.

Overview
=======

This project aims to implement and deliver a JDBC compliant Java driver project for exposing Ariel data via AQL queries, from a QRadar system.  The project is only compatible with QRadar v7.2.4+.

The usage of the project's built driver jar, is as per standard JDBC Java coded constructs; or with use in conjunction with SQL client tools or reporting engines which support custom drivers (e.g. Birt, Jasper, SqlSquirell, Crystal Reports ...)

These driver and project source code come without warranties of any kind. 

Any issues discovered using the samples should not be directed to QRadar support, but be reported on the Github issues tracker.  

The driver does not aim to be fully compliant to the JDBC specification; but aims to allow QRadar Advance Query Language (AQL) sql statements to be run against a QRadar system, to return readonly datasets; whilst progressively implement improved compliance against the JDBC spec.

Development notes
=============
## PRE-REQUISITES

1. git 1.7.1 (or later) - http://git-scm.com/downloads
2. maven 3.2.2 (or later) - http://maven.apache.org/download.cgi
3. Java JDK (1.7+) - http://www.oracle.com/technetwork/java/javase/downloads/jdk7-downloads-1880260.html

### CLONE:
```
git clone https://github.com/ibm-security-intelligence/jdbc-driver.git
```

### USEFUL COMMANDS:

#### 1) BUILD, (with no tests), PACKAGE the driver jar:
```
mvn -Dmaven.test.skip=true clean package
```


If you want to both build AND run the test suite, or portions of the test suite.  You will need to edit all the *Test.properties files under the <jdbc-driver>/src/test directory amending all the properties in each file to suite your QRadar environment.  The username/password combination must be a valid QRadar user, attached to a User Role, defined with both Log Activity and Network Activity privileges granted.  For testing convenience, use the QRadar admin user for connection.


```
ip=localhost
user=admin
password=password
url=jdbc:qradar://localhost/
```


#### 2) BUILD, Run ALL Tests, PACKAGE the driver jar:
```
mvn clean package
```

#### 2) BUILD, Run Specific Test 
```
mvn  -Dtest=SomeTestClass#someTestMethod test 
```

### OUTPUT

- <jdbc-driver>/target/jaql-0.1.jar
- <jdbc-driver>/target/jaql-0.1-jar-with-dependencies.jar

Use the resulting **jaql-0.1-jar-with-dependencies.jar** as your jdbc driver, with your reporting engine / SQL client of choice to connect to an Ariel datastore.

Usage
=====
Key notes for usage:

- **main Driver class**: `com.ibm.si.jaql.Driver`
- **url**: jdbc:qradar://_Qradar-Console_/
- **username**:  _admin-user_
- **password**:  _admin-user-password_

AQL SQL Syntax
==============
The publicly available documentation stack for IBM's Qradar Security Intelligence Platform, includes a reference section for Qradar's Ariel Database Query Language.

- http://www-01.ibm.com/support/knowledgecenter/SS42VS_7.2.4/com.ibm.qradar.doc_7.2.4/c_aql_intro.html?lang=en 

SparkSQL Support
================
SparkSQL supports the ability to directly query an SQL database using a JDBC driver and load the results into a DataFrame for further processing. This package has experimental support for identifying and converting Spark-generated SQL queries into valid AQL queries and handling nuances of the AQL REST interface when possible. In Spark, when reading `jdbc` formatted data, simply specify add `com.ibm.si.jaql.Driver` for the `driver`. 

### Optional
There is a second project, spark, that includes a JDBC Dialect for SparkSQL. To build it,
```
cd spark
sbt package
```
After that, you can register the dialect in Spark
```
import org.apache.spark.sql.jdbc.QRadarAqlDialect
import org.apache.spark.sql.jdbc.JdbcDialects
JdbcDialects.registerDialect(QRadarAqlDialect)
```
The dialect does not seem to be strictly required, as the changes made to the jdbc-driver appropriately converts the SQL into AQL. Works till needs to be done to map the types, but QRadar / AQL using the REST API may default to VARCHAR / Strings.

### Example
```scala
val dataframe_qradar = sqlContext.read.format("jdbc").option("url", "jdbc:qradar://127.0.0.1:443/").option("driver", "com.ibm.si.jaql.Driver").option("dbtable", "(SELECT sourceip,destinationip,username FROM events)").option("user", "admin").option("password", "password").load()
```
```
dataframe_qradar: org.apache.spark.sql.DataFrame = [sourceip: string, destinationip: string, username: string]
```
```scala
dataframe_qradar.show
```
```
+------------+-------------+--------------+
|    sourceip|destinationip|      username|
+------------+-------------+--------------+
|10.10.12.168|    127.0.0.1|          NULL|
|10.10.12.168|    127.0.0.1|          NULL|
|10.10.12.168| 10.10.12.168|configservices|
|10.10.12.168| 10.10.12.168|configservices|
|10.10.12.168| 10.10.12.168|configservices|
|10.10.12.168| 10.10.12.168|configservices|
|10.10.12.168| 10.10.12.168|configservices|
|10.10.12.168| 10.10.12.168|configservices|
|10.10.12.168| 10.10.12.168|configservices|
|10.10.12.168| 10.10.12.168|configservices|
|10.10.12.168| 10.10.12.168|configservices|
|10.10.12.168| 10.10.12.168|configservices|
|10.10.12.168| 10.10.12.168|configservices|
|10.10.12.168| 10.10.12.168|configservices|
|10.10.12.168|    127.0.0.1|          NULL|
|10.10.12.168|    127.0.0.1|          NULL|
|10.10.12.168|    127.0.0.1|          NULL|
|10.10.12.168|    127.0.0.1|          NULL|
|10.10.12.168|    127.0.0.1|          NULL|
|10.10.12.168|    127.0.0.1|          NULL|
+------------+-------------+--------------+
only showing top 20 rows
```
```scala
dataframe_qradar.groupBy("sourceip").count().show
+------------+-----+
|    sourceip|count|
+------------+-----+
|10.10.12.168|  356|
|   127.0.0.1|    2|
+------------+-----+
```