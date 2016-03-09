# Change Log
## [2.2rc4]
- Remove Warn and Crit information in performance data since nagios support only WARN and CRIT but script uses 4 values for this: minWARN, maxWARN, minCRIT and maxCRIT.

## [2.2rc3]
- Fix performance data function not work properly since it still using old configuration syntax

## [2.2rc2]
- Change line 247 to use old exception syntax

## [2.2rc1]
- Allow global check, script now return with exit code status follow by hardware status, but not prefix.

## [2.1]
### Release 4
- Fix bug not read state thresholds from configuration file

### Release 3
- Fix typo mistakes

### Fixed bug:
- Fix bug while check vdisk bad block

### Features:
- Change configuration file syntax
- Script no longer requires configuration file to work.
- You can use both cli and configuration file at same time. cli parameters will overwrite configurations. or you can set everything in configuraiton file.
- Disables alert no longer disable performance data.
- Disables alert now also remove prefix state string.
- No longer sets default theshold for hardware. State alert still get default threshold if not set.
- Allows ignore alert on chosen hardware by leaving its alert threshold blank or use 'none' string.
- Allows set threshold for every hardware at once in configuration file.
- Remove feature generate default configuration file. You should use the one on this git repo.

### Change code:
- Improve code structure.
- Remove manual function.

## [2.0b9]
### Fixed bug:
- Fix mistake when edit code in 2.0b8

## [2.0b8]
### Fixed bug:
- Remove duplicate snmp data when checking "PS"

## [2.0b7]
### Fixed bugs:
- Fix critical error in comparison alert value.
- Fix error while parsing Sensor threshold.

## [2.0b6]
### Fixed bugs:
- Fix error when using SNMPv2 for checking all hw.

## [2.0b5]
### Fixed bugs:
- Add code at line 876: hw_no_alert = config_check(hw_cfg, hw_no_alert) to allow check all HW at once when use config file.

## [2.0b4]
### Change code:
- Change "is False" instead of "is None"

## [2.0b3]
### Change code:
- Dump "(n/a)" to missing OID to avoid script crash. 

### Fixed bugs:
- Fix bug "required config file" when check Hardware group. 

## [2.0b2]
### Change codes:
- Improve speed.
- Use "snmpget" instead of "snmpwalk" which may cause SNMP time-out when multi check instances run. Scan mode still using "snmpwalk".
- Remove cached mode.

## [2.0b1]
### Fixed bugs:
- Fix critical bug in cached mode which may lead to hosts just read from one host's cache file.
- Fix "-C" error when use SNMPv2.

### Change codes:
- Allow input COMMUNITY STRING in command option but SNMPv3 still read from configuration file (for security purpose). When input COMMUNITY, script assumes you chose SNMPv2c.
SNMP:
- Only SNMPv2c and v3 accepted.
Features:
- Check all hardware or hardware group now requires only SNMP authentication info to run.

## [2.0b]

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

### [1.x]

- Fix some bugs.

### [1.0]

- First version.
