

		swRaidMIB

		Net-SNMP-Plugin for monitoring
		Linux RAID devices

		(c) Copyright G. Kuhlmann, 2007

		This is free software. See COPYING for license.




INTRODUCTION:
============

This software is a loadable plugin module for the Net-SNMP agent to monitor the software RAID functions of a Linux system.

It works by reading the file /proc/mdstat and providing the information read under the ucdExperimental OID with sub-ID 18. This sub-ID hasn't been registered with anyone yet, but it was free at the time this software was written.

The plugin module provides the following read-only variable tree:

````
+--swRaidMIB(18)
   |
   +--swRaidTable(1)
   |  |
   |  +--swRaidEntry(1)
   |     |  Index: swRaidIndex
   |     |
   |     +-- -R-- Integer32 swRaidIndex(1)
   |     |        Range: 0..65535
   |     +-- -R-- String    swRaidDevice(2)
   |     |        Textual Convention: DisplayString
   |     |        Size: 0..255
   |     +-- -R-- String    swRaidPersonality(3)
   |     |        Textual Convention: DisplayString
   |     |        Size: 0..255
   |     +-- -R-- String    swRaidUnits(4)
   |     |        Textual Convention: DisplayString
   |     |        Size: 0..255
   |     +-- -R-- Integer32 swRaidUnitCount(5)
   |     +-- -R-- EnumVal   swRaidStatus(6)
   |              Textual Convention: RaidStatusTC
   |              Values: inactive(1), active(2), faulty(3)
   |
   +-- -R-- Integer32 swRaidErrorFlag(100)
   +-- -R-- String    swRaidErrMessage(101)
            Textual Convention: DisplayString
            Size: 0..255
````

swRaidTable - Contains Information about every RAID device in the system.
              It consists of the following elements:
- swRaidIndex        -  Internal index number for each device
- swRaidDevice       -  Device name (for example "`/dev/md1`")
- swRaidPersonality  -  RAID personality of this device (for example "`Raid1`")
- swRaidUnits        -  Units within this RAID device encoded as a blank-seperated string (for example "`/dev/sda1 /dev/sdb1`")
- swRaidUnitCount    -  Number of units in the RAID device
- swRaidStatus       -  Status of the RAID device. This can be one of: `inactive`, `active`, `faulty`
- swRaidErrorFlag - Contains a `non-zero` value if any of the RAID devices on the  system is faulty
- swRaidErrMessage - Contains the names of all faulty RAID devices in case the `swRaidErrorFlag` is non-zero

The `swRaidStatus`, `swRaidErrorFlag` and `swRaidErrMessage` variables are primarily provided to
get a quick overview of the RAID system, for example by the use of an auto-
mated monitoring script. They can also be used to generate SNMP trap or inform
messages in case one or more of the RAID devices failed.



REQUIREMENTS:
============

To use this plugin module you obviously need a somewhat current Net-SNMP agent.

It has been tested with Net-SNMP version 5.4.
I don't know if it works with any older version. The agent has to be compiled with support
for loadable modules (which is the default). In addition the Net-SNMP runtime libraries and include files have to be installed in order to
compile the module.


INSTALLATION:
============

The swRaidMIB plugin module is not a huge software project, so no configure script has been written. Since this module only makes
sense on a Linux system, and gcc is usually installed on such a system, the Makefile has been written to only support the GNU development tools.

### Debian

1. Install dependencies on Debian: libsnmp-dev
```bash
sudo apt-get install libsnmp-dev
```

2. Check it out using Git
```bash
git clone git@github.com:opennms-config-modules/snmp-swraid.git
```

3. change directory and link net-snmp
```bash
cd snmp-swraid
ln -s /usr/include/net-snmp .
```


4. run "make" in the source directory.
```bash
make
```

5. copy the resulting file swRaidPlugin.so

 To a place where the SNMP agent can find it, for example /usr/lib/snmp/dlmod.

	```bash
	sudo cp -ar swRaidPlugin.so /usr/lib/snmp/dlmod
	```
6. Copy the MIB SWRAID-MIB.txt

	Copy file named SWRAID-MIB.txt to the place where all your other MIBfiles are stored, usually something like /usr/local/share/netsnmp/mibs or /usr/share/snmp/mibs.

	```bash
	sudo cp -ar SWRAID-MIB.txt /usr/share/snmp/mibs
	```
7. Modify   `/etc/snmp/snmpd.conf`

	In order for the SNMP agent to find the module you need a line in the snmpd configuration file telling it to load the module:

	```bash
	sudo nano /etc/snmp/snmpd.conf
	```
	add the following:
	```bash
	dlmod swRaidMIB /usr/lib/snmp/dlmod/swRaidPlugin.so
	```

8. Restart SNMPD service
```bash
systemctl restart snmpd
```


### Other Unix-like systems
To compile on other Unix-like systems you would probably need to modify the Makefile.

## testing the snmpget

If you like snmpwalk/get to load the mib by default edit `/etc/snmp/snmp.conf`

```
-m+SWRAID-MIB
```

which causes the agent to load the SWRAID-MIB in addition to all the other internal MIBs.

You can now view the RAID data
using your preferred SNMP client program, like mbrowse or snmpwalk.
Depending on your security policies you can try the following command:

```bash
snmpwalk -v1 -c public -m+SWRAID-MIB localhost swRaidMIB
```
This should show you the same information you can get from the `/proc/mdstat`
file.

The plugin module will re-read `/proc/mdstat` at most once every 60 seconds. If you want to test the module, set one of the RAID drives to a
faulty state (for example using the mdadm program) and wait a minute until
you can see the result through SNMP.

SUPPORT:
=======

If you need any help using this module, or you find a bug please let me
know. Also, if you can think of any improvement or even have implemented
any improvement yourself please send me an E-Mail to:

gero@gkminix.han.de

Please note that I wrote this program in my spare time, so answers may
sometimes take a bit longer. I will not offer any commercial support for
this program.
