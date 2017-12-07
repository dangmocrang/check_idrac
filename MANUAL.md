# Check_iDRAC version 2.1
Nguyen Duc Trung Dung (ndtdung@spsvietnam.vn - dung.nguyendt@gmail.com)

website: 

AWS Cloud Ops at ADURO, INC

#Manual 

## How it works?
- Simple, this check use snmpwalk command which included with net-snmp package you installed.
- It run snmpwalk instance to send SNMP request to idrac and parsing result data into human readable form.

## Requires:
- This check included with DELL IDRAC MIB file (required). You can copy MIB file to your default MIB folder, usually
is /usr/share/snmp/mibs/. If you don't want to do this, keep it where you want and use option -m/--mib with absolute path to
it.

## Supported hardware type:
- Power Supply
- Power Unit
- Fan
- Memory
- Battery CMOS
- Physical Disk
- Virtual Disk
- CPU
- Sensor
- Global

## How to use?
### Everything in cli
**Scan all**

./check_idrac_2.py -H 10.10.10.20 -v2c -c public

```PS
PS
--PS 1: OK, Volt I/O: 264 V/230 V, Current: 0.4 A, Watt I/O: 900 W/750 W
--PS 2: OK, Volt I/O: 264 V/230 V, Current: 0.2 A, Watt I/O: 900 W/750 W
FAN
--System Board Fan1: 2760 RPM - ENABLED/OK
--System Board Fan2: 2760 RPM - ENABLED/OK
--System Board Fan3: 2880 RPM - ENABLED/OK
--System Board Fan4: 2520 RPM - ENABLED/OK
--System Board Fan5: 2760 RPM - ENABLED/OK
--System Board Fan6: 2760 RPM - ENABLED/OK
BATTERY
--System Board CMOS Battery: ENABLED/OK [PRESENCEDETECTED]
--PERC1 ROMB Battery: ENABLED/OK [PRESENCEDETECTED]
--PERC2 ROMB Battery: ENABLED/OK [0]
PU
--PU 1: ENABLED/OK, RedundancyStatus: FULL, SystemBoard Pwr Consumption: 112 W
MEM
--Memory 1 (DIMM Socket A1) 16.00 GB/1600 MHz: ENABLED/OK [DDR3, Samsung, S/N: 36BDCC73]
--Memory 2 (DIMM Socket A2) 16.00 GB/1600 MHz: ENABLED/OK [DDR3, Samsung, S/N: 36BDCC8D]
--Memory 3 (DIMM Socket B1) 16.00 GB/1600 MHz: ENABLED/OK [DDR3, Samsung, S/N: 36BDCC8A]
--Memory 4 (DIMM Socket B2) 16.00 GB/1600 MHz: ENABLED/OK [DDR3, Samsung, S/N: 36BDCC09]
VDISK
--VDisk 1 (XXX): OK/ONLINE, RAID-5 (1675.12 GB), BadBlock: 0 [Virtual Disk 0 on Integrated RAID Controller 1]
DISK
--PDisk 1 (0:1:0) 558.38 GB: READY, PowerStat: SPUNUP, HotSpare: dedicated [WD, S/N: WXA1E63UYD10]
--PDisk 2 (0:1:1) 558.38 GB: ONLINE, PowerStat: SPUNUP, HotSpare: no [WD, S/N: WXA1E63UYC84]
--PDisk 3 (0:1:2) 558.38 GB: ONLINE, PowerStat: SPUNUP, HotSpare: no [WD, S/N: WXA1E63UXM21]
--PDisk 4 (0:1:3) 558.38 GB: ONLINE, PowerStat: SPUNUP, HotSpare: no [WD, S/N: WXA1E63UXX39]
--PDisk 5 (0:1:4) 558.38 GB: ONLINE, PowerStat: SPUNUP, HotSpare: no [WD, S/N: WXA1E63UXS14]
SENSOR
--System Board Inlet Temp: 22.0 C ENABLED/OK
--System Board Exhaust Temp: 30.0 C ENABLED/OK
--CPU1 Temp: 40.0 C ENABLED/OK
--CPU2 Temp: 41.0 C ENABLED/OK
CPU
--CPU 1 (8 cores/16 threads): ENABLED/OK [Intel(R) Xeon(R) CPU E5-2650 0 @ 2.00GHz]
--CPU 2 (8 cores/16 threads): ENABLED/OK [Intel(R) Xeon(R) CPU E5-2650 0 @ 2.00GHz]
```

**Group scan**

./check_idrac_2.py -H 10.10.10.20 -v2c -c public -w MEM

```
Memory 1 (DIMM Socket A1) 16.00 GB/1600 MHz: ENABLED/OK [DDR3, Samsung, S/N: 36BDCC73]
Memory 2 (DIMM Socket A2) 16.00 GB/1600 MHz: ENABLED/OK [DDR3, Samsung, S/N: 36BDCC8D]
Memory 3 (DIMM Socket B1) 16.00 GB/1600 MHz: ENABLED/OK [DDR3, Samsung, S/N: 36BDCC8A]
Memory 4 (DIMM Socket B2) 16.00 GB/1600 MHz: ENABLED/OK [DDR3, Samsung, S/N: 36BDCC09]
```

**Check specific hardware**

./idrac2.1.py -H 10.10.10.20 -v2c -c public -w MEM#3

```
OK - Memory 3 (DIMM Socket B1) 16.00 GB/1600 MHz: ENABLED/OK [DDR3, Samsung, S/N: 36BDCC8A]
```

***Everything in configuration file and for specific hardware***

./check_idrac_2.py -H 10.10.10.20 -f check_idrac.conf -w MEM#2

***Set alert threshold on Output WATT for Power Supply***

./idrac2.1.py -H 10.10.10.20 -v2c -c public -w PS#1 --wat-warn=none,750

```
WARN - PS 1: OK, Volt I/O: 264 V/0 V, Current: 0.4 A, Watt I/O: 900.0 W/750(!) W
```

***Set alert for Fan#1***

./idrac2.1.py -H 10.10.10.20 -v2c -c public -w FAN#1 --fan-warn=4000,6000 --fan-crit=3000,7000

***Set alert for Fan#1 in configuration file***

./idrac2.1.py -H 10.10.10.20 -v2c -c public -w FAN#1

within configuration file, under [fan] section:

```
[fan]
thesholds = 4000,6000,3000,7000
```

or you can combine both config file and command cli option

```
[fan]
thesholds = none,6000,3000,7000
```

./idrac2.1.py -H 10.10.10.20 -v2c -c public -w FAN#1 --fan-warn=4000

this is the same as above example

***Threshold explanation***

For some hardware types, you will see --xxx-warn=MINWARN,MAXWARN or MINCRIT,MAXCRIT. So what is this?

Let's assume we have a sensor temperature, if temperature raise too high it would be trouble and if temp is too low we are in serious trouble too, so to ensure the temp is not higher then 60 degree and now drop below 30 degree we define threshold in pair:
```
--temp-warn=29,61
--temp-crit=25,65
```
This means if temp lower than 30 degree (in range of 26 to 29 degree) OR if temp higher than 60 degree (in range of 61 to 64) check will show you WARNING.
If temp lower than 26 (in range of 25 or lower) OR if temp higher than 64 (in rage of 65 or higher) check will show you CRITICAL.

## What is STATE ALERT DEFINITION?
- Every admin has their own reason/style when checking device so I dont want to force you to expected
an alert like I do. That is reason for State alert and is defined in config file, that means
every hardware part which has status as: ok/online/ready/disable/enable... will have
alert rule based on these configs.
- As in sample config file you will see something like this:

```
OK = ok|online|spunup|full|ready|enabled|presence
WARN = $ALL$
CRIT = critical|nonRecoverable|fail
```

So in example, third RAM DIMM with output like this:
```
Memory 3 (DIMM Socket B1) 16.00 GB/1600 MHz: ENABLED/OK [DDR3, Samsung, S/N: 36BDCC8A]
```
If OK State was defined with: 
```
OK = ok|online|spunup|full|ready|enabled|presence
```
The check result will be:
```
OK - Memory 3 (DIMM Socket B1) 16.00 GB/1600 MHz: ENABLED/OK [DDR3, Samsung, S/N: 36BDCC8A]
```
You see the words: ENABLED/OK ? The prefix "OK" exist because we defined "OK = ok|online|spunup|full|ready|enabled|presence" in config. If you change config to:
```
OK = online|spunup|full|ready|presence
```
Check result will be this:
```
WARN - Memory 3 (DIMM Socket B1) 16.00 GB/1600 MHz: ENABLED(!)/OK(!) [DDR3, Samsung, S/N: 36BDCC8A]
```

Its because "WARN = $ALL$" so all status the out of "OK" and "CRIT" range will be WARN.
- You can either define state alert by options --ok/--warn/--crit or in config file.
- You can not use $ALL$ for two states at same time.
- I use simple search, in ex "online" will matchs onlines/xxonline/onlinebutnotok... but since device does not return
those weird outputs so we will be ok.
- If no $ALL$ in state definition, all states that are not belong to states we defined here, will be reported as UNKNOWN.

Why would someone need --no-alert?
- For Testing purpose, check will not ask for threshold and speed up analyzing processes.
- No alert also disables performance data output.

- "-n" option will bypass alert function (also not ask for threshold) and always return with exit code = 0. Be carefull
do not use "-n/--no-alert" options in production environment!!!
