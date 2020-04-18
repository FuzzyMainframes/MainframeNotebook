# Adding a DASD to MVS 3.8J
Information on adding another DASD to MVS.  This will not cover performing an IOGEN.  We will use the existing IOGEN.  

If an IOGEN is needed, see the IOGEN document [to be written].

> **NOTE**  
> The 3390-3 is the largest supported DASD, but it doesn’t support it very well. Even 3380s are not supported perfectly. 3350 is where MVS 3.8 wants to be ideally.
> 
## Hercules Configuration
### Tell Hercules about the DASD.

Edit the hercules configuration file for the mainframe.  In a TK4- install, this will be the conf/tk4-.cnf file.

TK4- disk images are named "volumne_name.device_address".  For example, MVSRES on device 148 is located in the file dasd/mvsres.148.

In the tk4-.cnf, DASDs are defined in the following format:
```
device_id device_type dasd_image
```

For example, the MVSRES volume, on a model 3350 DASD, and located as an image file dasd/mvsres.148 would look like this in the conf/tk4-.cnf file.
```
0148 3350 dasd/mvsres.148
```


#### Create the file to hold the DASD image
```bash
dasdinit -bz2 -a dasd/myvol.242 3350 MYVOL
```
Paramater | Definition
----------|-
-bz2      | designates this to be a compressed volume
-a        | instructs dasdinit to create alternate tracks .Most MVS utilities and programs will not function properly without the presence of the alternate tracks.
myvol.242 | filename of the DASD image on the Hercules host machine
3350      | Model of the DASD 
MYVOL     | Volume serial number as seen by MVS.

#### IPL the Mainframe
IPL the mainframe and wait for MVS to finsish loading,
```
./mvs
```
#### Take the volume offline
From the Hercules console vary the new drive offline (device id 242 in this example):
```
/v 242,offline
/s dealloc
```

#### Format the volume
Login to TSO and submit JCL to format the DASD:


```jcl
//HERC01  JOB (1),ICKDSF,CLASS=A,MSGCLASS=X
//ICKDSF EXEC PGM=ICKDSF,REGION=4096K
//SYSPRINT DD  SYSOUT=*
//SYSIN    DD  *
  INIT UNITADDRESS(<address>) NOVERIFY VOLID(<volser>) OWNER(HERCULES) -
               VTOC(0,1,<extent>)
/*
//
```
Variable | Definition
---------|-
address  | Device Address [242]
volser   | Volume Serial Number [MYVOL]
extent   | number of tracks to reserve for VTOC [40, should be fine for 3350]

Update the JCL to reflect the details of the DASD to format.  Also, update the MSGCLASS to H to hold the outpoot in the spool so that it can be reviewed.
```jcl
//HERC01  JOB (1),ICKDSF,CLASS=A,MSGCLASS=H
//ICKDSF EXEC PGM=ICKDSF,REGION=4096K
//SYSPRINT DD  SYSOUT=*
//SYSIN    DD  *
  INIT UNITADDRESS(242) NOVERIFY VOLID(MYVOL) OWNER(HERCULES) -
               VTOC(0,1,40)
/*
//
```

Back in the console, MVS will ask "ICK003D REPLY U TO ALTER VOLUME 242 CONTENTS, ELSE T".  

To confirm that you want to format the drive, enter reply in the console"
```
/r 00,U
```
Check the job outpoot and confirm that the RC=0.

#### Mount the volume.
```
/m 242,vol=(sl,MYVOL),USE=private
```
The volume should now be accessible at this point, but will we still need to make sure it is automatically mounted each time we IPL the mainfranme.

#### Edit SYS1.PARMLIB(VATLST00)
In RFE, edit PDS member SYS1.PARMLIB(VATLST00), adding a new record at the bottom of the list:

The parameters specified for each volume are:
Column(s) | Description
----------|-
1 - 6     | the volume serial number 
8         | 0 = permanently resident volumes; 1 = reserved volumes. Use 0 in most cases.
10        | The use attribute. 0 = STORAGE; 1 = PUBLIC; 2 = PRIVATE
12 - 19   |Denotes the device type.
20        | Just a comma ','
21        | indicates whether the operator should be requested to mount the volume if it is not found during IPL.  In most cases you should specify N (for NO) in this column.
23 - 71   | may be used for comments


# References
* [Adding a disk device to your MVS or z/OS system - M14](https://www.youtube.com/watch?v=UXCaXF0n0F4) - video by Moshix.  Note that he inadvertently leaves off the '-a' paramater in for the dasdinit command initially which causes some errors, causing him to spend some time troubleshooting the issue.  
* [Adding DASD Volumes](http://www.jaymoseley.com/hercules/installMVS/addingDasd.htm) - Jay Moseley's article on adding a DASD.  Good reference, but was written for Hercules 3.11.