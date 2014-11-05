Check_iDRAC
Version 2.0b4

Change log:
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
