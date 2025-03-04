# Measure query performance improvement

## Introduction

In this lab you will run some typical queries against the TimesTen cache and against the Oracle database to illustrate the performance benefit of TimesTen.

Estimated Time: **5 minutes**

**IMPORTANT:** As noted in the previous lab there are many factors that can affect performance. As a result, the performance numbers shown in this lab are indicative only. The numbers that _you_ measure will differ and may be slightly better or slightly worse.

When timing database query execution it is important to understand what you are timing, and to time the right things, otherwise you may get misleading results.

When a tool, such as SQL\*Plus or ttIsql, or an application submits a SQL statement to the database several things occur.

1.	The database will parse the SQL statement to make sure that it is a syntactically correct, valid SQL statement.
2.	The database query optimizer will prepare an optimal query execution plan for the statement.
3.	The prepared query plan will be executed.
4.	The results (result set) will be returned.
5.	The query tool will format and display the returned results.

Steps 1 and 2 are referred to as ‘preparing’ the query. Depending on the complexity of the query these steps may be quite costly.

Steps 3 and 4 represent the actual query execution, as we normally think of it.

Step 5 is important if we are interested in the results but is not relevant in terms of timing the query performance. However, this step can be very expensive, especially if the result set is large and/or is being displayed on a terminal, and so including it in the query timing will significantly distort the overall timing.

Another important consideration is that there will always be some variability in the time taken by a query due to other factors such as OS scheduling, system load, concurrency effects and so on. This is especially relevant if the query execution time is short. For this reason, one should always time several (ideally at least 10) executions of a query and take an average, rather than rely on timing just one or two executions.

Both SQL*Plus and ttIsql offer various features to allow you to time queries:

**SQL\*Plus**

-	The SET TIMING ON command will provide query timings with an accuracy of around 0.01 seconds.
-	Proper use of the autotrace functionality (set autotrace traceonly statistics) will suppress the formatting and display of the returned results.
-	There is no mechanism to separate out the prepare step from the query execution/fetch step.

**ttIsql**

-	The ‘timing on’ command provides operation timing (not just for SQL queries) with _microsecond_ accuracy.
-	The ‘verbosity 0’ command suppresses the formatting and display of the returned results.
-	Explicit support, via the ‘prepare’ and ‘execandfetch’ commands, for separating the prepare and execute/fetch steps.

So it is possible to perform basic query timing with both SQL\*Plus and ttIsql. However, the limited timing granularity of SQL*Plus means that it is not that useful for queries that run in a very short amount of time.

A much better way to time queries, with any database, is to use a program written for the purpose. Such a program (**queryTimer**) is available to you along with an associated convenience script (**timeQueries**) which will run each of the queries in an input file multiple times (10 in this case), time just the execute/fetch step and then report the average. The syntax for using the script is as follows:

`usage: timeQueries { -oracle | -timesten } <filename>`

where ‘filename’ is the name of a file containing the query text. The source code for the queryTimer utility is available in the main VM in the directory **~/lab/src**. Here is the timeQueries script:

```
#!/bin/bash

declare -r queryTimer=/tt/livelab/bin/queryTimer
declare -r qtparams="-user oe -pwd oe -count 10 -verbose -service"
declare -r oraservice="orclpdb1"
declare -r ttservice="sampledb"

declare -i ret=0

usage()
{
    echo "usage: timeQueries { -oracle | -timesten } <filename>"
    exit 100
}

if [[ $# -ne 2 ]]
then
    usage
fi

case "$1" in
    "-oracle")
        ${queryTimer} ${qtparams} ${oraservice} "$2"
        ret=$?
        ;;
    "-timesten")
        ${queryTimer} ${qtparams} ${ttservice} "$2"
        ret=$?
        ;;
    *)
        usage
        ;;
esac

exit ${ret}
```

We will use this script to investigate the performance of three different queries running against both the Oracle database and the TimesTen cache.

### Objectives

- Run queries against Oracle database and measure the elapsed times.
- Run the same queries against TimesTen and measure the elapsed times.
- Compare the query times.

### Prerequisites

This lab assumes that you have:

- Completed all the previous labs in this workshop, in sequence.

## Task 1: Connect to the environment

If you do not already have an active terminal session, connect to the OCI compute instance and open a terminal session, as the user **oracle**. In that terminal session, connect to the TimesTen host (tthost1) using ssh.

## Task 2: Examine the test queries

Examine the three queries that we will use for this exercise:

**cat ~/queries/query_1.sql**

```
[oracle@tthost1 livelab]$ cat ~/queries/query_1.sql
SELECT
    c.customer_id,
    c.cust_first_name,
    c.cust_last_name,
    c.phone_number,
    o.order_id,
    o.order_date,
    o.order_status,
    o.order_total
  FROM
    customers c INNER JOIN orders o ON c.customer_id = o.customer_id
WHERE
    c.customer_id IN (104, 108, 144)
ORDER BY 6;
```
**cat ~queries/query_2.sql**

```
[oracle@tthost1 livelab]$ cat ~queries/query_2.sql
SELECT
    o.order_id,
    oi.line_item_id,
    oi.product_id,
    pi.product_name,
    pi.product_description,
    (oi.unit_price * oi.quantity)
  FROM
    customers c,
    orders o,
    order_items oi,
    product_information pi
 WHERE
    (c.customer_id = 144) and
    (c.customer_id = o.customer_id) and
    (o.order_id = oi.order_id) and
    (oi.product_id = pi.product_id)
 ORDER BY 1, 2;
```
**cat ~queries/query_3.sql**

``` 
[oracle@tthost1 livelab]$ cat ~queries/query_3.sql
SELECT
    i.product_id,
    i.warehouse_id,
    i.quantity_on_hand,
    pi.product_name,
    pi.product_description
  FROM
    inventories         i,
    product_information pi
 WHERE
    (i.product_id = pi.product_id) AND
    (pi.product_status = 'orderable')
 ORDER BY 1, 3 DESC;
```
 
There is also a file, **queries/query_all.sql**, that contains all three queries.

## Task 3: Run the queries against the Oracle database

First run the queries against the Oracle database:

**~/bin/timeQueries -oracle queries/query_all.sql**

```
[oracle@tthost1 livelab]$ ~/bin/timeQueries -oracle queries/query_all.sql
info: connected to 'orclpdb1' (Oracle Database 19c Enterprise Edition Release 19.0.0.0.0 - Production)
info: running queries from file 'queries/query_all.sql'
info: ========================================
info: executing query #1
info: ----------------------------------------
SELECT
    c.customer_id,
    c.cust_first_name,
    c.cust_last_name,
    c.phone_number,
    o.order_id,
    o.order_date,
    o.order_status,
    o.order_total
  FROM
    customers c INNER JOIN orders o ON c.customer_id = o.customer_id
 WHERE
    c.customer_id IN (104, 108, 144)
 ORDER BY 6
info: ----------------------------------------
info: query #1 average execution + fetchall time (rowcount = 13) = 416.0 us
info: ========================================
info: executing query #2
info: ----------------------------------------
SELECT
    o.order_id,
    oi.line_item_id,
    oi.product_id,
    pi.product_name,
    pi.product_description,
    (oi.unit_price * oi.quantity)
  FROM
    customers c,
    orders o,
    order_items oi,
    product_information pi
 WHERE
    (c.customer_id = 144) and
    (c.customer_id = o.customer_id) and
    (o.order_id = oi.order_id) and
    (oi.product_id = pi.product_id)
 ORDER BY 1, 2
info: ----------------------------------------
info: query #2 average execution + fetchall time (rowcount = 45) = 1633.0 us
info: ========================================
info: executing query #3
info: ----------------------------------------
SELECT
    i.product_id,
    i.warehouse_id,
    i.quantity_on_hand,
    pi.product_name,
    pi.product_description
  FROM
    inventories         i,
    product_information pi
 WHERE
    (i.product_id = pi.product_id) AND
    (pi.product_status = 'orderable')
 ORDER BY 1, 3 DESC
info: ----------------------------------------
info: query #3 average execution + fetchall time (rowcount = 1112) = 25688.2 us
info: ========================================
info: disconnected from 'orclpdb1'
```

## Task 4: Run the queries against the TimesTen cache

Now run the queries against the TimesTen cache:

**~/bin/timeQueries -timesten queries/query_all.sql**

```
[oracle@tthost1 livelab]$ ~/bin/timeQueries -timesten queries/query_all.sql
info: connected to 'sampledb' (Oracle TimesTen IMDB version 22.1.1.3.0)
info: running queries from file 'queries/query_all.sql'
info: ========================================
info: executing query #1
info: ----------------------------------------
SELECT
    c.customer_id,
    c.cust_first_name,
    c.cust_last_name,
    c.phone_number,
    o.order_id,
    o.order_date,
    o.order_status,
    o.order_total
  FROM
    customers c INNER JOIN orders o ON c.customer_id = o.customer_id
 WHERE
    c.customer_id IN (104, 108, 144)
 ORDER BY 6
info: ----------------------------------------
info: query #1 average execution + fetchall time (rowcount = 13) = 34.2 us
info: ========================================
info: executing query #2
info: ----------------------------------------
SELECT
    o.order_id,
    oi.line_item_id,
    oi.product_id,
    pi.product_name,
    pi.product_description,
    (oi.unit_price * oi.quantity)
  FROM
    customers c,
    orders o,
    order_items oi,
    product_information pi
 WHERE
    (c.customer_id = 144) and
    (c.customer_id = o.customer_id) and
    (o.order_id = oi.order_id) and
    (oi.product_id = pi.product_id)
 ORDER BY 1, 2
info: ----------------------------------------
info: query #2 average execution + fetchall time (rowcount = 45) = 114.0 us
info: ========================================
info: executing query #3
info: ----------------------------------------
SELECT
    i.product_id,
    i.warehouse_id,
    i.quantity_on_hand,
    pi.product_name,
    pi.product_description
  FROM
    inventories         i,
    product_information pi
 WHERE
    (i.product_id = pi.product_id) AND
    (pi.product_status = 'orderable')
 ORDER BY 1, 3 DESC
info: ----------------------------------------
info: query #3 average execution + fetchall time (rowcount = 1112) = 1819.8 us
info: ========================================
info: disconnected from 'sampledb'
```

## Task 4: Compare the results

Here is a comparion of the results that I obtained when I ran these queries (your results _will_ differ):


| Query |	Average Oracle Query Time (us) | Average TimesTen Query Time (us) | TimesTen Speedup |
| :---- | :----------------------------- | :------------------------------- | :------------ |
| 1 | 416.0 | 34.2 | 12.1x |
| 2 | 1633.0 | 114.0 | 14.3x |
| 3 | 25688.2 | 1819.8 | 14.1x|


You can now *proceed to the next lab*. Keep your primary terminal session open for use in the next lab.

## Acknowledgements

* **Author** - Chris Jenkins, Senior Director, TimesTen Product Management
* **Contributors** -  Doug Hood & Jenny Bloom, TimesTen Product Management
* **Last Updated By/Date** - Chris Jenkins, July 2022

