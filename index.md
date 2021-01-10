# Oracle 19c Installation and Configuration

I recently started installing and configuring Oracle database on my local windows machine. 

I am trying to list down few steps that may help you to setup your oracle database.

## Pre-requisite

* The system must have at least 8 GB of RAM. 
  
  As windows 10 OS takes at least 3-4 GB of RAM and Oracle installation needs 3-4 GB of RAM, you must have at least 8 GB of RAM on the system.
  
* The system must have 10GB of free space

## First Step: Download Oracle Setup

* Create profile at oracle.com, if you do not have already.

* Download Oracle Database - Download the latest oracle 19c installation from oracle website. 
https://www.oracle.com/ca-en/database/technologies/oracle19c-windows-downloads.html#license-lightbox/

### Few things to remember before installation:

* Create windows user before starting installation

  During the installation, setup will ask you to create a windows user with non-administrative privileges. Good practice is to create the windows user before hand.
  
* Do not use @ in your oracle user password
  
  When you connect to oracle database, you use @ symbol in the connection string. for e.g. sqlplus sys/<passowrd>@orcl as sysdba. if there is @ in your password, you will get   errors.

* check your environment variable for TNS_ADMIN otherwise create TNS_ADMIN environment variable and set the path

* Do not select Container Database, install only single database

* Rest of the setup is straight forward. It may take 20-30 minutes.

## Second Step: Database Configuration 

Real challenge is database configuration. Let me list down down few of my learnings:

There are 2 types of Database that can be installed during oracle 19c setup, Container and Pluggable DB. If you have not changed anything in your oracle installation then by default the container DB is ORCL and pluggable database is ORCLPDB. 

If you just need single database then do not select Container database during installation

In order to connect to the database, you need to configure listener.ora and tnsnames.ora files. You must read the basic concepts of both of them.

I assume that you have installed only single database.

Oracle installation will create listener and tnsnames files for you at the TNS_ADMIN path location. But the setup does not create another setting for listener in the listener.ora file that you have to create it. This you will learn later.

### 1 TNS Names

#### 1.1 Creating entry in tnsnames.ora for listener 


* check your environment variable for TNS_ADMIN otherwise create TNS_ADMIN environment variable and set the path

* LISTENER_ORCL should already be there in tnsnames.ora file

* if not, check local_listener in the database to create the listener in tnsnames.ora

```
c:\ sqlplus sys as sysdba

SQL> show parameter local_listener;

NAME                                 TYPE        VALUE

local_listener                       string      LISTENER_ORCL
```

* Create the entry in tnsnames.ora file, if does not exist

```
LISTENER_ORCL =
  (ADDRESS = (PROTOCOL = TCP)(HOST = localhost)(PORT = 1521))
```

#### 1.2 Entry in tnsnames.ora for tnsname - 


* Entry for database (ORCL) must have been created during the installation but do double check.

* SERVICE_NAME must match the instance name.

* Check the instance name in the database

```
c:\ sqlplus sys as sysdba

SQL> show parameter instance_name;

NAME                                 TYPE        VALUE

instance_name                        string      orcl
```


* Create the below entry in tnsnames.ora, ORCL must have been already there

```
ORCL =
  (DESCRIPTION =
    (ADDRESS = (PROTOCOL = TCP)(HOST = localhost)(PORT = 1521))
    (CONNECT_DATA =
      (SERVER = DEDICATED)
      (SERVICE_NAME = orcl)
    )
  )
 
```

### 2 Entry in listener.ora. 

* SID_NAME must match with tnsname (orcl and orclpdb) that we created in tnsnames.ora file

```
SID_LIST_LISTENER =
  (SID_LIST =
    (SID_DESC =
      (SID_NAME = CLRExtProc)
      (ORACLE_HOME = E:\oracle\oracle_install)
      (PROGRAM = extproc)
      (ENVS = "EXTPROC_DLLS=ONLY:E:\oracle\oracle_install\bin\oraclr19.dll")
    )
	(SID_DESC =
      (SID_NAME = orcl)
      (ORACLE_HOME = E:\oracle\oracle_install)
    )
  )

LISTENER =
  (DESCRIPTION_LIST =
    (DESCRIPTION =
      (ADDRESS = (PROTOCOL = TCP)(HOST = localhost)(PORT = 1521))
      (ADDRESS = (PROTOCOL = IPC)(KEY = EXTPROC1521))
    )
  )

```

### 3 Check host file on system (C:\Windows\System32\drivers\etc)

* localhost name resolution is handled within DNS itself. UnComment localhost entry

```
127.0.0.1 localhost
```

### Restart the system


Voila!
