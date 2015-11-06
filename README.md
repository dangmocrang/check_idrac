# Check_iDRAC

Version 2.2rc1

**Configuration file syntax change since version 2.1**

[CHANGELOG] (./CHANGELOG.md)

## Features
Check idrac 7 hardwares status via SNMP. Currently supports these hardware:
- Virtual Disk
- Physical Disk
- Memory
- CPU
- Power Supply
- Power Unit
- Fan
- Battery
- Temperature Sensor

## Example
### Example 1

./idrac_2.0b8 -H 10.10.10.1 -f check_idrac.conf -w sensor

```
System Board Inlet Temp: 20.0 C ENABLED/OK

System Board Exhaust Temp: 30.0 C ENABLED/OK

CPU1 Temp: 40.0 C ENABLED/OK

CPU2 Temp: 41.0 C ENABLED/OK.
```

### Example 2

./idrac_2.0b8 -H 10.10.10.1 -f check_idrac.conf -w sensor#1

```
OK - System Board Inlet Temp: 19.0 C ENABLED/OK
```

## Manual
[MANUAL] (./MANUAL.md)

## License

[GPLv3] (./LICENSE.md)
