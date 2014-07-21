SmartMON
========

`Author` https://www.facebook.com/mrsunlin

`Wiki` https://github.com/mrsunlin/SmartMON/wiki

`Website` http://mrsunlin.github.io/SmartMON

# Introduction
SmartMON is a Unix/Linux Infrastructure Monitor, all built on Standard Shell scripts. 

Already included powerful monitor functions for Oracle  9i/10g/11g databases and RACs. 

The whole system has been tested and smoothly monitored a very large Telecom Product System for several years , which have hundreds hosts and hundreds TB data stored in Oracle Database.

# Deployment
It's very simple to deploy the product when the environment is prepared, only one line code in the crontab:

    crontab -e
    * * * * * sh <Path to the monitor directory>/smartmon_agent

## Requirement
### Hardware
One PC Server is OK for monitoring one hundred of machines.
### Software
`OS:` Linux/Unix with FTP service

`Database:` Oracle (9i or above)
### Network
Only two ports of the server should be open to all of the monitored machines:

> `21`, `1521`

## Define the parameters
### Define Services to be monitored
edit file `cmd.cfg`, define available services. Normally you won't need more than the default.

Each service is defined by 2 lines:

    # tabdef;<service name>;<col count>;<if partitioned by date 1 else 0>;<if partitioned by houer 1 else 0>;<partition count>;[<col name>=<col type]; ...]
    # <service name>;<platform name, * for all platform>;<execution interval in seconds>;<col count>;<group name>;<command>

The first line define the table structure and data life cycle in database. The` <col count>` here should equal with `<col count>` in the command definition line, that's for the safe check. The `<col type>` is exactly the type in Oracle database.

The second line define the execution time cycle, and the process group. `<execution interval in seconds>` define the time cycle in seconds, if less than one minute, the actual interval is one minute. That's for the safety concern.

`<group name>` define which process should the service use on the monitored machine, if multiple services share the same `<group name>`, they will be executed sequently in the same process, otherwise they will be executed separately. 

`<command>` define the command/Script to be executed by the service on the monitored machine. For simple command, it can be embedded in the `<command>`. For complex service which need to be done with scripts, the scripts should be stored in `scripts` directory, and in the `<command>` should have a line of command like `sh scripts/some_script_name`

### Define machines to be monitored
edit file `service.cfg`, each line defines a machine and the services on it.

The format is `hostname service1;service2;...;serviceN`

### Define the database
In `db_conn.cfg` file define two database users, they can reside in the same database or separately. 

The `alert` database only stores the data no more than 2 hours, so that the alert system can detect events as soon as possible.

The `report` database stores data as much as the definition of services specified by `<partition count>`, but drop older data. That enable you analyze history events across machines in one pass.

Here is the example

    alert	sqlplus -s smartmon_alert/smartmon12@"(DESCRIPTION=(ADDRESS=(HOST=10.19.84.10)(PORT=1521)(PROTOCOL=TCP))(CONNECT_DATA=(SID=ols1)))"
    report	sqlplus -s smartmon_report/smartmon12@"(DESCRIPTION=(ADDRESS=(HOST=10.19.84.10)(PORT=1521)(PROTOCOL=TCP))(CONNECT_DATA=(SID=ols1)))"

You can use external database(s), but you should make sure the monitored machines can access the `1521` port (Oracle Listener port) of the database(s)

### Define the release path
The release path is defined in file `release_server.cfg`. The content contains only one line:

    ftp <ip address> 21 <smartmon deployed os username> <password> <release path>

the `<release path>` is a directory path which can be accessed by the smartmon OS user via FTP. 

It is for every monitored machine to download the latest monitor package, to update their local definitions or scripts in no more than 1 minute when new package is released.

When anyone of the files with suffix `.cfg` or any script file in `scripts` directory is modified, simply execute command `sh release_agent` to distribute the package to all monitored machines.

# Event alert & analyze
This part is not finished yet
## Alert
Override the file `smartmon_alert` to make your own alert system

## Analyze
Currently, use SQL tools to access the `report` database to do analyze manually.

# More
It's open to you to add more functions to the system, such as alert, analyze etc.
