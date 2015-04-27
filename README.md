Check_iDRAC
Version 2.0b7

This check monitor idrac 7 hardware status via SNMP:
- Virtual Disk
- Physical Disk
- Memory
- CPU
- Power Supply
- Power Unit
- Fan
- Battery
- Temperature Sensor

This script query SNMP directly to IDRAC device, not via any openmanage suit.
Example:
./idrac_2.0b7 -H 10.10.10.1 -f check_idrac.conf -w sensor
System Board Inlet Temp: 20.0 C ENABLED/OK
System Board Exhaust Temp: 30.0 C ENABLED/OK
CPU1 Temp: 40.0 C ENABLED/OK
CPU2 Temp: 41.0 C ENABLED/OK


Change log:
2.0b7
- Fix critical error in comparison alert value.
- Fix error while parsing Sensor threshold.

2.0b6:
- Fix error when using SNMPv2 for checking all hw.

2.0b5:
- Add code at line 876: hw_no_alert = config_check(hw_cfg, hw_no_alert) to allow check all HW at once when use config file.

2.0b4:
- Change "is False" instead of "is None"

2.0b3:
- Dump "(n/a)" to missing OID to avoid script crash. 
- Fix bug "required config file" when check Hardware group. 

2.0b2:
- Improve speed.
- Use "snmpget" instead of "snmpwalk" which may cause SNMP time-out when multi check instances run. Scan mode still using "snmpwalk".
- Remove cached mode.

2.0b1:
Bugs:
- Fix critical bug in cached mode which may lead to hosts just read from one host's cache file.
- Fix "-C" error when use SNMPv2.
Options:
- Allow input COMMUNITY STRING in command option but SNMPv3 still read from configuration file (for security purpose). When input COMMUNITY, script assumes you chose SNMPv2c.
SNMP:
- Only SNMPv2c and v3 accepted.
Features:
- Check all hardware or hardware group now requires only SNMP authentication info to run.

2.0b:
- Rewrite code from thrash.
- Support more check: Virtual Disk, Battery, System Board Pwr Consumption.
- Re-format output.
- Power Supply and Power Unit now separated.
- Power Supply got Watt Input/Output information.
- FanUnit removed. Fan check now show details of every Fan.
- No need to use IPMI to check Fan rpm/Power Volt, Current.
- SNMP authentication (v2/v2c/v3) now read only from config file.
- MIB can now load from specific file. Default will load ALL Mibs.
- Add function to generate sample config file.
- Allow user to define their own state alert.
- Clear code and guide comment for anyone who want to make this script unique.

1.x:
- Fix some bugs.

1.0:
- First version.
