# Load Oracle data into the cache

## Introduction

In this lab you will load data from the Oracle tabls into the TimesTen cache tables. This action will also activate the AUTOREFRESH mechanism which will periodically refresh the cache with any changes that have occurred in the Oracle database.

Estimated Time: **5 minutes**

### Objectives

- Load the APPUSER and OE cache groups

This task is accomplished using SQL statements, so can be easily performed from application code if required.

### Prerequisites

This lab assumes that you have:

- Completed all the previous labs in this workshop, in sequence.

## Task 1: Connect to the environment

If you do not already have an active terminal session, connect to the OCI compute instance and open a terminal session, as the user **oracle**.

In that terminal session, connect to the TimesTen host (tthost1) using ssh.

## Task 2: Load the APPUSER cache group

As you saw in the previous lab, when a READONLY cache group is first created its tables are empty and the autorefresh mechanism is in a paused state.

Loading the cache group populates the cache tables with the data from the Oracle database and also activates the autorefresh mechanism. The load occurs in such a manner that if any changes occur oin Oracle while the load is in progress those changes will be captured and then autorefreshed to TimesTen ince the load is completed.

Load the APPUSER.CG\_VPN\_USERS cache group (1 million rows) and then examine the cach group and table:

**ttIsql "DSN=sampledb;UID=appuser;PWD=appuser;OraclePWD=appuser"**

```
oracle@tthost1 livelab]$ ttIsql "DSN=sampledb;UID=appuser;PWD=appuser;OraclePWD=appuser"
Copyright (c) 1996, 2022, Oracle and/or its affiliates. All rights reserved.
Type ? or "help" for help, type "exit" to quit ttIsql.

connect "DSN=sampledb;UID=appuser;PWD=********;OraclePWD=********";
Connection successful: DSN=sampledb;UID=appuser;DataStore=/tt/db/sampledb;DatabaseCharacterSet=AL32UTF8;ConnectionCharacterSet=AL32UTF8;LogFileSize=256;LogBufMB=256;PermSize=1024;TempSize=256;OracleNetServiceName=ORCLPDB1;
(Default setting AutoCommit=1)
Command>
```
**LOAD CACHE GROUP appuser.cg\_vpn\_users COMMIT EVERY 1024 ROWS;**

```
Command> LOAD CACHE GROUP appuser.cg_vpn_users COMMIT EVERY 1024 ROWS;
1000000 cache instances affected.
```
**cachegroups cg\_vpn\_users;**

```
Command> cachegroups cg_vpn_users;

Cache Group APPUSER.CG_VPN_USERS:

  Cache Group Type: Read Only
  Autorefresh: Yes
  Autorefresh Mode: Incremental
  Autorefresh State: On
  Autorefresh Interval: 2 Seconds
  Autorefresh Status: ok
  Aging: No aging defined

  Root Table: APPUSER.VPN_USERS
  Table Type: Read Only

1 cache group found.
```
**tables;**

```
Command> tables;
  APPUSER.VPN_USERS
1 table found. 
```
**select count(\*) from vpn\_users;**

```
Command> select count(*) from vpn_users;
< 1000000 >
1 row found.
```
Update optimizer statistics for all the tables in the APPUSER schema:

**statsupdate;**

```
Command> statsupdate;
```
Exit from ttIsql:

**quit**

```
Command> quit
Disconnecting...
Done.
```

Note that the status of autorefresh has now changed to  **On**.

```
Autorefresh State: On
```

## Task 3: Load the OE cache groups

Now do the same for the OE schema cache groups:

**ttIsql "DSN=sampledb;UID=oe;PWD=oe;OraclePWD=oe"**

```
[oracle@tthost1 livelab]$ ttIsql \
    "DSN=sampledb;UID=oe;PWD=oe;OraclePWD=oe"
Copyright (c) 1996, 2022, Oracle and/or its affiliates. All rights reserved.
Type ? or "help" for help, type "exit" to quit ttIsql.

connect "DSN=sampledb;UID=oe;PWD=********;OraclePWD=********";
Connection successful: DSN=sampledb;UID=oe;DataStore=/tt/db/sampledb;DatabaseCharacterSet=AL32UTF8;ConnectionCharacterSet=AL32UTF8;LogFileSize=256;LogBufMB=256;PermSize=1024;TempSize=256;OracleNetServiceName=ORCLPDB1;
(Default setting AutoCommit=1)
Command> 
```
**LOAD CACHE GROUP oe.cg\_promotions COMMIT EVERY 1024 ROWS;**

```
Command> LOAD CACHE GROUP oe.cg_promotions COMMIT EVERY 1024 ROWS;
2 cache instances affected.
```
**LOAD CACHE GROUP oe.cg\_prod\_inventory COMMIT EVERY 1024 ROWS;**

```
Command> LOAD CACHE GROUP oe.cg_prod_inventory COMMIT EVERY 1024 ROWS;
288 cache instances affected.
```
**LOAD CACHE GROUP oe.cg\_cust\_orders COMMIT EVERY 1024 ROWS;**

```
Command> LOAD CACHE GROUP oe.cg_cust_orders COMMIT EVERY 1024 ROWS;
319 cache instances affected.
```
**statsupdate;**

```
Command> statsupdate;
```
**tables;**

```
Command> tables;
  OE.CUSTOMERS
  OE.INVENTORIES
  OE.ORDERS
  OE.ORDER_ITEMS
  OE.PRODUCT_DESCRIPTIONS
  OE.PRODUCT_INFORMATION
  OE.PROMOTIONS
7 tables found.
```
**select count(\*) from customers;**

```
Command> select count(*) from customers;
< 319 >
1 row found.
```
**select count(\*) from inventories;**

```
Command> select count(*) from inventories;
< 1112 >
1 row found.
```
**select count(\*) from orders;**

```
Command> select count(*) from orders;
< 105 >
1 row found.
```
**select count(\*) from order\_items;**

```
Command> select count(*) from order_items;
< 665 >
1 row found.
```
**select count(\*) from product\_descriptions;**

```
Command> select count(*) from product_descriptions;
< 8639 >
1 row found.
```
**select count(\*) from product\_information;**

```
Command> select count(*) from product_information;
< 288 >
1 row found.
```
**select count(\*) from promotions;**

```
Command> select count(*) from promotions;
< 2 >
1 row found.
```
**quit**

```
Command> quit
Disconnecting...
Done.
```

You can now *proceed to the next lab*. Keep your terminal session open for use in the next lab.

## Acknowledgements

* **Author** - Chris Jenkins, Senior Director, TimesTen Product Management
* **Contributors** -  Doug Hood & Jenny Bloom, TimesTen Product Management
* **Last Updated By/Date** - Chris Jenkins, July 2022

