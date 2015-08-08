# Check_iDRAC
Version 2.0b9

[CHANGELOG.md] (./CHANGELOG.md)

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

## Example 1

./idrac_2.0b8 -H 10.10.10.1 -f check_idrac.conf -w sensor

```
System Board Inlet Temp: 20.0 C ENABLED/OK

System Board Exhaust Temp: 30.0 C ENABLED/OK

CPU1 Temp: 40.0 C ENABLED/OK

CPU2 Temp: 41.0 C ENABLED/OK.
```

## Example 2

./idrac_2.0b8 -H 10.10.10.1 -f check_idrac.conf -w sensor#1

```
OK - System Board Inlet Temp: 19.0 C ENABLED/OK
```
