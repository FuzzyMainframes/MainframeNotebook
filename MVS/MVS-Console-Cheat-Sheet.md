# MVS Console Cheat Sheet <!-- omit in toc -->
*written by [roblthegreat](https://github.com/roblthegreat)*

This document is designed to be a quick reference guide of the more commonly used MVS console commands.  This guide targets MVS 3.8J TK4- users, though most would apply to any version of MVS.

> **NOTE** 
> 
>All commands listed here are prefixed with `/` which is required if the commands are entered from the Hercules console.  If you are using an alternate console, you must omit the `/`.

## Contents <!-- omit in toc -->
- [MVS](#mvs)
  - [Force logout of hung user session (cancel user)](#force-logout-of-hung-user-session-cancel-user)
  - [DASDs and Volumes](#dasds-and-volumes)
    - [Take a volume offline](#take-a-volume-offline)
    - [Mount a Volume](#mount-a-volume)
    - [Display List of all ONLINE DASDs](#display-list-of-all-online-dasds)
    - [Display List of all defined DASDs and their status](#display-list-of-all-defined-dasds-and-their-status)
- [JES2](#jes2)
  - [COLD start the JES2 spool](#cold-start-the-jes2-spool)
  - [Clear JES2 spool](#clear-jes2-spool)
  - [Display JES2 spool utilization](#display-jes2-spool-utilization)
  - [Display the list of jobs in the JES2 spool](#display-the-list-of-jobs-in-the-jes2-spool)
- [VTAM](#vtam)
  - [Start/stop hung VTAM terminal](#startstop-hung-vtam-terminal)
- [RAKF](#rakf)
  - [Activate RAKF User Table Changes](#activate-rakf-user-table-changes)
  - [Activate RAKF Profile Table Changes](#activate-rakf-profile-table-changes)
- [FTPD](#ftpd)
  - [Start FTPD](#start-ftpd)
  - [Stop FTPD](#stop-ftpd)
- [NJE38](#nje38)


# MVS

## Force logout of hung user session (cancel user)  
```
/cancel u={userid} 
```
## DASDs and Volumes
### Take a volume offline
```
/v {dev_id},offline
/s dealloc
```

### Mount a Volume
```
/v {dev_id},online
/m {dev_id},vol=(sl,{vol_ser}),USE={use_class}
```

> #### Storage Use Classes
> 
> A DASD volume is always mounted with one of the following use attributes:
> Class   | Use
> --------|---
> `PRIVATE` | New datasets will be created only if the volume serial is specified
> `PUBLIC`  | MVS may place a temporary dataset on this volume, if volume serial is not specified.  
> `STORAGE` | MVS places permanent datasets on this volume if the user did not supply a volume serial


### Display List of all ONLINE DASDs
```
/d u,dasd,online
```
### Display List of all defined DASDs and their status
```
/d u,dasd
```


# JES2

## COLD start the JES2 spool
The first command stops JES2 and the second re-starts it with a COLD start request.  This will remove anything in JES2 spool.
```
/$pjes2
/$sjes2,parm='COLD,NOREQ'
```
## Clear JES2 spool
```
/$PJ1-9999
/$PT1-9999 
/$PS1-9999
```

##  Display JES2 spool utilization
```
/$d n,all
```

## Display the list of jobs in the JES2 spool
```
/$d j1-9999 
/$d s1-9999
/$d t1-9999
```
# VTAM
## Start/stop hung VTAM terminal
```
/v net,inact,id=cuu{terminal_address}
/v net,act,id=cuu{terminal_address}
```
# RAKF
## Activate RAKF User Table Changes
```
/S RAKFUSER
```
## Activate RAKF Profile Table Changes
```
/S RAKPROF
```

# FTPD
## Start FTPD
FTPD is disabled by default.
```
/START FTPD,SRVPORT={port_num}
```
{port_num} is the TCP port that will be bound
## Stop FTPD
```
/c ftpd
```

# NJE38