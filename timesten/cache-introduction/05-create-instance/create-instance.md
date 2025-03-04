# Create a TimesTen instance

## Introduction

In this lab you will create a TimesTen instance to host our TimesTen cache database, start the instance and execute a few simple TimesTen commands.

Estimated Time: 5 minutes.

### Objectives

- Create a TimesTen instance
- Start the instance
- Execute a few basic TimesTen commands

### Prerequisites

This lab assumes that you have:

- Completed all the previous labs in this workshop, in sequence.

## Task 1: Connect to the environment

If you do not already have an active terminal session, connect to the OCI compute instance and open a terminal session, as the user **oracle**.

## Task 2: Connect to the TimesTen host

Connect to the TimesTen host (tthost1) using ssh:

**ssh tthost1**

```
[oracle@ttlivelabvm ~] ssh tthost1
Your current directory is:  /tt/livelab
[oracle@tthost1 livelab]$
```
Take a look at the directory contents:

**ls -l**

```
[oracle@tthost1 livelab]$ ls -l
total 16
drwxr-xr-x. 2 oracle oinstall   22 May 26 13:10 bin
drwxr-xr-x. 2 oracle oinstall 4096 May 26 13:10 queries
drwxr-xr-x. 2 oracle oinstall 4096 May 26 13:10 scripts
-rw-r--r--. 1 oracle oinstall  316 May 10 12:55 tables_appuser.sql
-rw-r--r--. 1 oracle oinstall 3879 May 10 14:31 tables_oe.sql
```

## Task 3: Create a TimesTen instance

A TimesTen _installation_ is comprised of the TimesTen software components. An installation is created by unzipping the TimesTen software distribution media into a suitable location. For this workshop, the TimesTen software distribution media has already been unzipped into the directory **/shared/sw** to create a TimesTen installation named **tt22.1.1.3.0**. Take a look at that:

**ls -l /shared/sw**

```
[oracle@tthost1 livelab]$ ls -l /shared/sw
total 0
dr-xr-x---. 17 oracle oinstall 277 May  5 22:20 tt22.1.1.3.0
```

**ls -l /shared/sw/tt22.1.1.3.0**

```
[oracle@tthost1 livelab]$ ls -l /shared/sw/tt22.1.1.3.0
total 108
dr-xr-x---. 3 oracle oinstall    89 May  5 22:20 3rdparty
dr-xr-x---. 2 oracle oinstall  4096 May  5 22:19 bin
dr-xr-x---. 4 oracle oinstall    31 May  5 22:19 grid
dr-xr-x---. 3 oracle oinstall   240 May  5 22:19 include
dr-xr-x---. 2 oracle oinstall   167 May  5 22:19 info
dr-xr-x---. 2 oracle oinstall    26 May  5 22:19 kubernetes
dr-xr-x---. 3 oracle oinstall  4096 May  5 22:19 lib
dr-xr-x---. 3 oracle oinstall    19 May  5 22:19 network
dr-xr-x---. 3 oracle oinstall    18 May  5 22:19 nls
dr-xr-x---. 2 oracle oinstall   242 May  5 22:19 oraclescripts
dr-xr-x---. 4 oracle oinstall    40 May  5 22:20 PERL
dr-xr-x---. 7 oracle oinstall    68 May  5 22:19 plsql
-r--r-----. 1 oracle oinstall 99660 May  5 22:19 README.html
dr-xr-x---. 2 oracle oinstall    54 May  5 22:19 startup
dr-xr-x---. 2 oracle oinstall    90 May  5 22:19 support
dr-xr-x---. 3 oracle oinstall    54 May  5 22:20 ttoracle_home

```

You can create one or more TimesTen _instances_ from an installation. A TimesTen instance consists of various configuration files, log files and other things that together allow you to create and manage TimesTen databases. An instance is linked to the installation used to create it, so the installation must not be removed, renamed or modified in any way otherwise the operation of all linked instances will be affected.

When it is operational, a TimesTen instance also includes a set of associated processes which cooperate to manage the TimesTen databases that are owned by the instance.

Create a TimesTen instance called **ttinst** by using the **ttInstanceCreate** command located in the installation’s **bin** directory:

**/shared/sw/tt22.1.1.3.0/bin/ttInstanceCreate -location /tt/inst -name ttinst -tnsadmin /shared/tnsadmin**

```
[oracle@tthost1 livelab]$ /shared/sw/tt22.1.1.3.0/bin/ttInstanceCreate -location /tt/inst -name ttinst -tnsadmin /shared/tnsadmin

Creating instance in /tt/inst/ttinst ...

NOTE: The TimesTen daemon startup/shutdown scripts have not been installed.

The startup script is located here :
	'/tt/inst/ttinst/startup/tt_ttinst'

Run the 'setuproot' script :
	/tt/inst/ttinst/bin/setuproot -install
This will move the TimesTen startup script into its appropriate location.

The 22.1 Release Notes are located here :
  '/shared/sw/tt22.1.1.3.0/README.html'

Instance created successfully.

```

Copy the predefined **sys.odbc.ini** configuration file (more on that later) to the instance:

**cp scripts/sys.odbc.ini /tt/inst/ttinst/conf/sys.odbc.ini**

```
[oracle@tthost1 livelab]$ cp scripts/sys.odbc.ini /tt/inst/ttinst/conf/sys.odbc.ini
```

## Task 4: Start the instance

Whenever you work with TimesTen, it is _essential_ that you have the correct environment settings for the instance that you are working with. The easiest and safest way to do this is to source the environment file provided within the instance:

**source /tt/inst/ttinst/bin/ttenv.sh**

```
[oracle@tthost1 livelab]$ source /tt/inst/ttinst/bin/ttenv.sh
NOTE: TNS_ADMIN is already set in environment - /shared/tnsadmin

LD_LIBRARY_PATH set to /tt/inst/ttinst/install/lib:/tt/inst/ttinst/install/ttoracle_home/instantclient

PATH set to /tt/inst/ttinst/bin:/tt/inst/ttinst/install/bin:/tt/inst/ttinst/install/ttoracle_home/instantclient:/tt/inst/ttinst/install/ttoracle_home/instantclient/sdk:.:/home/oracle/bin:/usr/java/default/bin:/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin

CLASSPATH set to /tt/inst/ttinst/install/lib/ttjdbc8.jar:/tt/inst/ttinst/install/lib/orai18n.jar:/tt/inst/ttinst/install/lib/timestenjmsxla.jar:/tt/inst/ttinst/install/3rdparty/jms1.1/lib/jms.jar:.

TIMESTEN_HOME set to /tt/inst/ttinst
```

**Note:** The value of the **TIMESTEN_HOME** environment variable determines which TimesTen instance you are working with. 

Start the instance (i.e. start the main daemon) so that the instance is usable:

**ttDaemonAdmin -start**

```
[oracle@tthost1 livelab]$ ttDaemonAdmin -start
TimesTen Daemon (PID: 706, port: 6624) startup OK.
```

You now have an operational TimesTen instance that can host TimesTen databases.

## Task 5: View the database configuration file (sys.odbc.ini)

Examine the database configuration file sys.odbc.ini. This file is the main configuration file for the instance and defines all the databases that will be managed by the instance along with parameters used for connecting to them:

**cat $TIMESTEN_HOME/conf/sys.odbc.ini**

```
[oracle@tthost1 livelab]$ cat $TIMESTEN_HOME/conf/sys.odbc.ini
[ODBC Data Sources]
sampledb=TimesTen 22.1 Driver
sampledbcs=TimesTen 22.1 Client Driver

[sampledb]
DataStore=/tt/db/sampledb
PermSize=1024
TempSize=256
LogBufMB=256
LogFileSize=256
DatabaseCharacterSet=AL32UTF8
ConnectionCharacterSet=AL32UTF8
OracleNetServiceName=ORCLPDB1

[sampledbcs]
TTC_SERVER_DSN=SAMPLEDB
TTC_SERVER=tthost1-ext/6625
ConnectionCharacterSet=AL32UTF8
```

The file defines two ODBC Data Source Names (DSNs), **sampledb** and **sampledbcs**. ODBC is TimesTen’s native API, though TimesTen also provides, or supports, many other commonly used database APIs such as JDBC, Oracle Call Interface, ODP.NET, cx_Oracle (for Python) and node-oracledb (for Node.js).

The **sampledb** DSN is a _direct mode_, or _server_, _DSN_. It defines the parameters and connectivity for a database hosted by this TimesTen instance. Tools, utilities, and applications running on this host (tthost1) can connect via this DSN using TimesTen’s low latency ‘direct mode’ connectivity mechanism. This database is also accessible remotely using TimesTen’s client-server connectivity.

The **sampledbcs** DSN is a _client DSN_. It defines connectivity parameters for a server DSN which tools, utilities and applications can connect to using TimesTen’s client-server connectivity mechanism. In this example the DSN defines client-server access for the local sampledb server DSN.

All TimesTen APIs support both direct mode and client-server and, with some minor exceptions, the functionality is identical regardless of the type of connectivity that you are using.

## Task 6: Run some simple TimesTen commands

One of the simplest TimesTen utilities is **ttVersion**. This provides basic information about the TimesTen instance:

**ttVersion**

```
[oracle@tthost1 livelab]$ ttVersion
TimesTen Release 22.1.1.3.0 (64 bit Linux/x86_64) (ttinst:6624) 2022-05-05T19:45:28Z
  Instance admin: oracle
  Instance home directory: /tt/inst/ttinst
  Group owner: oinstall
  Daemon home directory: /tt/inst/ttinst/info
  PL/SQL enabled.
```
  
Another TimesTen utility is **ttStatus**, which displays information about the instance’s processes and any databases that it hosts:

**ttStatus**

```
[oracle@tthost1 livelab]$ ttStatus
TimesTen status report as of Thu May 26 14:02:13 2022

Daemon pid 706 port 6624 instance ttinst
TimesTen server pid 713 started on port 6625
------------------------------------------------------------------------
------------------------------------------------------------------------
Accessible by group oinstall
End of report
```

Currently there is not much to observe other than the process ids and port numbers used by the instance Daemon and Server processes.

You can now *proceed to the next lab*. Keep your terminal session open for use in the next lab.

## Acknowledgements

* **Author** - Chris Jenkins, Senior Director, TimesTen Product Management
* **Contributors** -  Doug Hood & Jenny Bloom, TimesTen Product Management
* **Last Updated By/Date** - Chris Jenkins, July 2022

