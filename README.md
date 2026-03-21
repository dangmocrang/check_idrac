# Check iDRAC

**Version 2.3.0** — Nagios/Icinga monitoring plugin for Dell iDRAC 7–10 hardware status monitoring via SNMP

**Author**: Nguyen Duc Trung Dung
**License**: GPLv3
**Status**: Production-ready

## Overview

`check_idrac` is a lightweight, single-file Python 3.11+ monitoring plugin that remotely checks Dell iDRAC 7–10 hardware status via SNMP. It integrates seamlessly with Nagios/Icinga to monitor:

- **Physical Components**: Memory, CPU, Power Supply, Fan, Battery, Temperature Sensor, Physical Disk
- **Virtual Components**: Virtual Disk
- **System Health**: Global system status

Key features:
- Dual alert system (state-based + threshold-based)
- SNMPv2c and SNMPv3 authentication support (SHA-384/SHA-512 for iDRAC 10)
- Flexible configuration (CLI args and/or INI config file)
- Nagios/Icinga standard output format with performance data

## Quick Start

### Prerequisites

```bash
# Install net-snmp (provides snmpwalk, snmpget)
# Requires net-snmp >= 5.8 for SHA-2/AES-256 (iDRAC 10 SNMPv3)
apt-get install snmp         # Debian/Ubuntu
yum install net-snmp          # RHEL/CentOS

# Ensure iDRAC-SMIv2.mib is in standard location
cp iDRAC-SMIv2.mib /usr/share/snmp/mibs/
```

### Basic Usage

```bash
# Check all hardware (SNMPv2c)
./check_idrac -H 10.10.10.20 -v 2c -c public

# Check specific hardware type (Memory)
./check_idrac -H 10.10.10.20 -v 2c -c public -w MEM

# Check specific device (Fan #1)
./check_idrac -H 10.10.10.20 -v 2c -c public -w FAN#1

# SNMPv3 with auth + privacy (iDRAC 7/8/9)
./check_idrac -H 10.10.10.20 -v 3 -u drac_user \
  -l authPriv -a sha -A authpass -x aes -X privpass

# SNMPv3 with SHA-512 + AES-256 (iDRAC 10)
./check_idrac -H 10.10.10.20 -v 3 -u drac_user \
  -l authPriv -a sha-512 -A authpass -x aes-256 -X privpass

# With config file
./check_idrac -H 10.10.10.20 -f check_idrac.conf

# With thresholds
./check_idrac -H 10.10.10.20 -v 2c -c public \
  --fan-warn=4000,6000 --fan-crit=3000,7000

# Output performance data for graphing
./check_idrac -H 10.10.10.20 -v 2c -c public -p

# Debug output
./check_idrac -H 10.10.10.20 -v 2c -c public -d
```

## Documentation

Full documentation available in `docs/`:

| File | Purpose |
|------|---------|
| [`docs/project-overview-pdr.md`](./docs/project-overview-pdr.md) | Project overview, requirements, stakeholders |
| [`docs/codebase-summary.md`](./docs/codebase-summary.md) | Repository structure, architecture, data flow |
| [`docs/code-standards.md`](./docs/code-standards.md) | Code conventions, naming, patterns, error handling |
| [`docs/system-architecture.md`](./docs/system-architecture.md) | Detailed architecture, component interactions, diagrams |
| [`docs/project-roadmap.md`](./docs/project-roadmap.md) | Current status, future enhancements, roadmap |
| [`MANUAL.md`](./MANUAL.md) | User manual with detailed examples |
| [`CHANGELOG.md`](./CHANGELOG.md) | Version history |

## Configuration

### INI Config File (Optional)

Sample config provided in `check_idrac.conf`:

```ini
[host]
ipaddress = 10.10.10.20

[snmp]
version = 2c
community = public

[fan]
thresholds = 4000,6000,3000,7000

[fan1]
# Override for specific fan
thresholds = 4500,5500,3500,6500

[sensor]
thresholds = 29,61,25,65

[state]
OK = ok|online|enabled|spunup|full|ready|presence
WARN = $ALL$
CRIT = critical|nonRecoverable|fail
```

**Notes**:
- CLI arguments override config file values
- Per-device overrides via `[hardware#number]` sections (e.g., `[fan1]`, `[watt1]`)
- Threshold format: `minWARN,maxWARN,minCRIT,maxCRIT`
- Use `none` to disable specific threshold boundary

## Hardware Types Supported

| Code | Hardware | Metrics |
|------|----------|---------|
| **GLOBAL** | Global System Status | System state |
| **MEM** | Memory | Size, speed, status |
| **CPU** | Processor | Cores, threads, brand, status |
| **PS** | Power Supply | Voltage I/O, current, wattage |
| **PU** | Power Unit | Status, redundancy, board consumption |
| **FAN** | Cooling/Fan | RPM, status, location |
| **SENSOR** | Temperature | Reading (°C), location, status |
| **BATTERY** | Battery (CMOS/ROMB) | Status, presence |
| **DISK** | Physical Disk | Capacity, state, power status, hot spare |
| **VDISK** | Virtual Disk | RAID level, capacity, state, bad blocks |

## CLI Options Reference

```
Required:
  -H, --host HOST          iDRAC IP address or hostname

SNMP Authentication:
  -v, --snmp-version {2c,3}         SNMP version (required)
  -c, --community COMMUNITY         SNMPv2c community string
  -u, --username USERNAME           SNMPv3 username
  -l, --security-level LEVEL        SNMPv3: authNoPriv|authPriv|noAuthNoPriv
  -a, --auth-protocol PROTOCOL      SNMPv3: sha|md5
  -A, --auth-password PASSWORD      SNMPv3: authentication passphrase
  -x, --privacy-protocol PROTOCOL   SNMPv3: aes|des
  -X, --privacy-password PASSWORD   SNMPv3: privacy passphrase

Configuration:
  -f, --config FILE        INI config file path
  -m, --mib FILE          Custom MIB file path

Filtering:
  -w, --hardware TYPE      Check only specific hardware (e.g., FAN, MEM#2)

Thresholds:
  --fan-warn MIN,MAX       Fan RPM warning threshold
  --fan-crit MIN,MAX       Fan RPM critical threshold
  --temp-warn MIN,MAX      Temperature warning threshold (°C)
  --temp-crit MIN,MAX      Temperature critical threshold (°C)
  --volt-warn MIN,MAX      Voltage warning threshold (V)
  --volt-crit MIN,MAX      Voltage critical threshold (V)
  --cur-warn MIN,MAX       Current warning threshold (A)
  --cur-crit MIN,MAX       Current critical threshold (A)
  --wat-warn MIN,MAX       Wattage warning threshold (W)
  --wat-crit MIN,MAX       Wattage critical threshold (W)
  --pwr-alert MIN,MAX      Power consumption warning
  --vdisk-bad MIN,MAX      Virtual disk bad blocks threshold

State Alerts:
  --ok PATTERN             Regex pattern for OK state
  --warn PATTERN           Regex pattern for WARN state
  --crit PATTERN           Regex pattern for CRIT state

Other:
  -n, --no-alert           Disable alerting (always exit 0)
  -p, --perf-data          Output performance data for graphing
  -d, --debug              Enable debug output
  --mem-modules N          Expected memory module count

Examples:
  ./check_idrac -H 10.10.10.20 -v 2c -c public
  ./check_idrac -H iDRAC1 -v 2c -c public -w FAN
  ./check_idrac -H iDRAC1 -v 2c -c public --fan-warn=4000,6000
  ./check_idrac -H iDRAC1 -f /etc/check_idrac.conf -p
  ./check_idrac -H iDRAC1 -v 3 -u user -l authPriv -a sha -A pass -x aes -X pass
```

## Alert System

### State-Based Alerts

Status strings are matched against regex patterns (OK/WARN/CRIT). Example:

```ini
OK = ok|online|enabled|spunup|full|ready|presence
WARN = $ALL$          # Anything not in OK or CRIT
CRIT = critical|nonRecoverable|fail
```

Special `$ALL$` wildcard matches any unclassified status.

### Threshold-Based Alerts

Numeric metrics checked against min/max pairs:

```
Threshold: minWARN,maxWARN,minCRIT,maxCRIT

Example: --temp-warn=29,61 --temp-crit=25,65
  WARN if temperature < 30 OR > 60
  CRIT if temperature < 26 OR > 64

Use 'none' to disable a boundary:
  --fan-warn=4000,none   WARN if RPM < 4000 only
```

## Output Format

**Status Line**:
```
OK - FAN
WARN - SENSOR: Excessive temperature on CPU1
CRIT - VDISK: Bad blocks detected
UNKNOWN - GLOBAL: SNMP query failed
```

**Device Details**:
```
--Fan1: 2760 RPM - OK
--Fan2: 3500(!) RPM - WARNING [threshold: 4000 RPM min]
--Fan3: 500(!!) RPM - CRITICAL [threshold: 3000 RPM min]
```

**Alert Markers**:
- `(!)` = WARNING
- `(!!)` = CRITICAL
- `(?)` = UNKNOWN

**Performance Data** (with `-p` flag):
```
OK - FAN|fan1_rpm=2760;4000;3000;0;5000 fan2_rpm=3500;4000;3000;0;5000
```

## Exit Codes

| Code | Meaning | Nagios Status |
|------|---------|--------------|
| 0 | All hardware OK | OK |
| 1 | Some hardware WARNING | WARNING |
| 2 | Some hardware CRITICAL | CRITICAL |
| 3 | Parse error, SNMP timeout, UNKNOWN state | UNKNOWN |

## Configuration File Syntax

See `check_idrac.conf` for complete example.

**Key Sections**:
- `[host]` — iDRAC IP/hostname
- `[snmp]` — SNMP version, authentication
- `[state]` — State alert patterns (OK/WARN/CRIT)
- `[fan]`, `[voltage]`, `[current]`, `[sensor]`, `[watt]`, `[consumption]`, `[vdisk]` — Global thresholds
- `[fan1]`, `[fan2]`, etc. — Per-device overrides

**Threshold Format**: `minWARN,maxWARN,minCRIT,maxCRIT`

## Nagios Integration

### Service Definition

```cfg
define service {
  service_description  Check iDRAC Hardware
  host_name            iDRAC1
  check_command        check_idrac!2c!public
  check_interval       5
  retry_interval       1
  max_check_attempts   3
  notification_period  24x7
}
```

### Command Definition

```cfg
define command {
  command_name    check_idrac
  command_line    /usr/lib/nagios/plugins/check_idrac \
                  -H $HOSTADDRESS$ \
                  -v $ARG1$ \
                  -c $ARG2$ \
                  -p
}
```

### With Config File

```cfg
define command {
  command_name    check_idrac_conf
  command_line    /usr/lib/nagios/plugins/check_idrac \
                  -H $HOSTADDRESS$ \
                  -f /etc/check_idrac.conf \
                  -w $ARG1$ \
                  -p
}

# Usage:
check_command check_idrac_conf!FAN,MEM,PS
```

## Troubleshooting

### SNMP Timeout

**Symptom**: `UNKNOWN - SNMP query timeout`

**Solutions**:
- Verify iDRAC is accessible: `snmpwalk -v 2c -c public 10.10.10.20 systemStateGlobalSystemStatus`
- Increase Nagios timeout in service definition: `check_timeout 30`
- Check network connectivity: `ping -c 3 10.10.10.20`
- Verify SNMP access: Check firewall rules on iDRAC

### Authentication Failed

**Symptom**: `UNKNOWN - SNMP query failed`

**Solutions**:
- SNMPv2c: Verify community string in config/CLI
- SNMPv3: Check username, auth protocol, auth password, privacy protocol, privacy password
- Test with net-snmp directly:
  ```bash
  snmpwalk -v 3 -u myuser -l authPriv -a sha -A authpass \
    -x aes -X privpass 10.10.10.20 systemStateGlobalSystemStatus
  ```

### Missing MIB File

**Symptom**: Numeric OID output instead of names

**Solutions**:
- Copy iDRAC-SMIv2.mib to `/usr/share/snmp/mibs/`
- Or specify path: `./check_idrac -H 10.10.10.20 -m /path/to/iDRAC-SMIv2.mib`

### No Data Returned

**Symptom**: `UNKNOWN - No data from SNMP query`

**Solutions**:
- Enable debug: `./check_idrac -H 10.10.10.20 -v 2c -c public -d`
- Verify iDRAC has been initialized (new hardware)
- Check iDRAC SNMP settings: Enable SNMP, allow v2c/v3

## Performance

- **Execution time**: 4-5 seconds per full check (10 hardware types)
- **Single device check**: ~1 second
- **Memory**: ~10MB resident
- **Network**: Single SNMP query per hardware type (no polling)

## Requirements

- **Python**: 3.11 or higher
- **net-snmp**: snmpwalk, snmpget binaries
- **iDRAC**: Version 7+ (tested on R640, R650 servers)
- **Network**: SNMP access to iDRAC (UDP port 161)

## Installation

```bash
# Copy plugin
cp check_idrac /usr/lib/nagios/plugins/
chmod 755 /usr/lib/nagios/plugins/check_idrac

# Copy config (optional)
cp check_idrac.conf /etc/
chmod 600 /etc/check_idrac.conf

# Copy MIB
cp iDRAC-SMIv2.mib /usr/share/snmp/mibs/

# Test
/usr/lib/nagios/plugins/check_idrac -H 10.10.10.20 -v 2c -c public
```

## Support & Contributing

- **Issues**: Report on GitHub or email author
- **Requests**: Describe use case and iDRAC model
- **Contributing**: Pull requests welcome (GPLv3 license)

**Author**: Nguyen Duc Trung Dung (ndtdung@spsvietnam.vn)

## License

GPLv3 — See LICENSE.md for details

## See Also

- [User Manual](./MANUAL.md)
- [Changelog](./CHANGELOG.md)
- [Documentation](./docs/)
