#!/usr/bin/env python
__author__ = 'Nguyen Duc Trung Dung'
__contact__ = 'ndtdung@spsvietnam.vn - dung.nguyendt@gmail.com'
__blog__ = 'dybn.blogspot.com'
__version__ = '2.0b7'
__license__ = 'GPL'

import commands
import sys
import optparse
import re
import os
run = commands.getstatusoutput

#----------------------DEFINE CONFIG DICT----------------------#
conf = {'snmp_version': None,
        'snmp_community': None,
        'snmp_v3_user': None,
        'snmp_v3_security_level': None,
        'snmp_v3_authentication_protocol': None,
        'snmp_v3_authentication_password': None,
        'snmp_v3_privacy_protocol': None,
        'snmp_v3_privacy_password': None,
        'snmp_binary': None,
        'fan_min_warn': None,
        'fan_max_warn': None,
        'fan_min_crit': None,
        'fan_max_crit': None,
        'ps_vol_min_warn': None,
        'ps_vol_max_warn': None,
        'ps_vol_min_crit': None,
        'ps_vol_max_crit': None,
        'ps_cur_min_warn': None,
        'ps_cur_max_warn': None,
        'ps_cur_min_crit': None,
        'ps_cur_max_crit': None,
        'ps_watt_min_warn': None,
        'ps_watt_max_warn': None,
        'ps_watt_min_crit': None,
        'ps_watt_max_crit': None,
        'pwr_consume_warn': None,
        'pwr_consume_crit': None,
        'sensor_temp_min_warn': None,
        'sensor_temp_max_warn': None,
        'sensor_temp_min_crit': None,
        'sensor_temp_max_crit': None,
        'vdisk_bad_warn': None,
        'vdisk_bad_crit': None,
        'ok': None,
        'warn': None,
        'crit': None}


#----------------------DEFINE HARDWARE DICT----------------------#
all_hardware = {
    #'DEV': ['SNMP name', 'Description/Additional SNMP name', 'Full name', [value/search pattern]]
    #Script will first read SNMP name to get snmp data form device, then use value part to search and parse info
    'MEM': ['memoryDeviceTable', '1.3.6.1.4.1.674.10892.5.4.1100.50',
            'Memory', r'memoryDeviceIndex\.|'
                      r'memoryDeviceStateSettings\.|'
                      r'memoryDeviceStatus\.|'
                      r'memoryDeviceType\.|'
                      r'memoryDeviceLocationName\.|'
                      r'memoryDeviceSize\.|'
                      r'memoryDeviceSpeed\.|'
                      r'memoryDeviceManufacturerName\.|'
                      r'memoryDeviceSerialNumberName\.'],
    'PS': ['powerSupplyTable', ['amperageProbeReading', 'voltageProbeReading'],  # PwrSupply, amperage Probe
           'PS', r'powerSupplyIndex\.|'
                 r'powerSupplyStatus\.|'
                 r'powerSupplyOutputWatts\.|'
                 r'powerSupplyInputVoltage\.|'
                 r'powerSupplyRatedInputWattage\.|'
                 r'amperageProbeReading\.|'
                 r'voltageProbeReading\.'],
    'PU': ['powerUnitTable', ['amperageProbeLocationName', 'amperageProbeReading'],  # PwrUnit, SystemBoard Pwr Consumption
           'PU', r'powerUnitIndex\.|'
                 r'powerUnitStateSettings\.|'
                 r'powerUnitRedundancyStatus\.|'
                 r'powerUnitStatus\.|'
                 r'amperageProbeLocationName\.|'
                 r'amperageProbeReading\.'],
    'CPU': ['processorDeviceTable', '1.3.6.1.4.1.674.10892.5.4.1100.30',
            'CPU', r'processorDeviceIndex\.|'
                   r'processorDeviceStateSettings\.|'
                   r'processorDeviceStatus\.|'
                   r'processorDeviceCoreCount\.|'
                   r'processorDeviceThreadCount\.|'
                   r'processorDeviceBrandName\.'],
    'SENSOR': ['temperatureProbeTable', 'Temperature Sensors',
               'Sensor', r'temperatureProbeIndex\.|'
                         r'temperatureProbeStateSettings\.|'
                         r'temperatureProbeStatus\.|'
                         r'temperatureProbeReading\.|'
                         r'temperatureProbeLocationName\.'],
    'FAN': ['coolingDeviceTable', 'Cooling Device',  # Fan status
            'Fan', r'coolingDeviceIndex\.|'
                   r'coolingDeviceStateSettings\.|'
                   r'coolingDeviceStatus\.|'
                   r'coolingDeviceReading\.|'
                   r'coolingDeviceLocationName\.'],
    'BATTERY': ['systemBattery', '',
                'Battery', r'systemBatteryIndex\.|'
                           r'systemBatteryStateSettings\.|'
                           r'systemBatteryStatus\.|'
                           r'systemBatteryReading\.|'
                           r'systemBatteryLocationName\.'],
    'DISK': ['physicalDiskTable', 'Physical Disk',  # Physical Disk
             'PDisk', r'physicalDiskNumber\.|'
                      r'physicalDiskName\.|'
                      r'physicalDiskManufacturer\.|'
                      r'physicalDiskState\.|'
                      r'physicalDiskSerialNo\.|'
                      r'physicalDiskCapacityInMB\.|'
                      r'physicalDiskSpareState\.|'
                      r'physicalDiskPowerState\.'],
    'VDISK': ['virtualDiskTable', 'Virtual Disk',
              'VDisk', r'virtualDiskNumber\.|'
                       r'virtualDiskName\.|'
                       r'virtualDiskState\.|'
                       r'virtualDiskSizeInMB\.|'
                       r'virtualDiskLayout\.|'
                       r'virtualDiskComponentStatus\.|'
                       r'virtualDiskBadBlocksDetected\.|'
                       r'virtualDiskDisplayName\.']}


#----------------------OPTION READER----------------------#
def option_reader():
    optp = optparse.OptionParser()
    optp.add_option('--man', help='show manual', action='store_true', dest='man')
    optp.add_option('-H', '--host', help='IP address', dest='host')
    optp.add_option('-c', '--community', help='SNMPv2 community', dest='community')
    optp.add_option('-f', '--conf', help='configuration file. Required for SNMPv3 authentication info', dest='cfg')
    optp.add_option('-m', '--mib', help='mib file. Default is ALL', dest='mib')
    optp.add_option('-n', '--no-alert', help='ignore alert threshold. Script will always return OK', action='store_true', dest='no_alert')
    optp.add_option('-w', '--hardware', help='hardware to check. If no hardware specified, all will be listed: DISK, VDISK, FAN, SENSOR, CPU, PS, PU, MEM, BATTERY', dest='hardware')
    optp.add_option('-p', '--perf', help='enable performance data. -n will toggling off this option.', dest='perf', action='store_true')
    optp.add_option('--fan-alert', help='FAN min_warn,min_crit', dest='fan_alert')
    optp.add_option('--temp-warn', help='TEMPERATURE min_warn,max_warn', dest='temp_warn')
    optp.add_option('--temp-crit', help='TEMPERATURE min_crit,max_crit', dest='temp_crit')
    optp.add_option('--volt-warn', help='OUT-VOLT min_warn,max_warn', dest='volt_warn')
    optp.add_option('--volt-crit', help='OUT-VOLT min_crit,max_crit', dest='volt_crit')
    optp.add_option('--cur-warn', help='OUT-CURRENT min_warn,max_warn', dest='cur_warn')
    optp.add_option('--cur-crit', help='OUT-CURRENT min_crit,max_crit', dest='cur_crit')
    optp.add_option('--wat-warn', help='OUT-WATT min_warn,max_warn', dest='wat_warn')
    optp.add_option('--wat-crit', help='OUT-WATT min_crit,max_crit', dest='wat_crit')
    optp.add_option('--pwr-alert', help='POWER CONSUME warn,crit. --pwr-alert=896,980', dest='consume')
    optp.add_option('--vdisk-bad', help='VDISK BAD Block warn,crit. --vdisk-bad=5,10', dest='vdiskbad')
    optp.add_option('--ok', help='OK state', dest='ok')
    optp.add_option('--warn', help='WARN state', dest='warn')
    optp.add_option('--crit', help='CRIT state', dest='crit')
    optp.add_option('--generate-config', help='generate default config file', action='store_true', dest='generate')
    opts, args = optp.parse_args()
    if opts.man is True:
        manual()
        sys.exit(0)
    if opts.generate is True:
        generate_config()
        sys.exit(0)
    if opts.host is None:
        print 'ERROR - Missing IP address'
        optp.print_help()
        sys.exit(1)
    else:
        if opts.community:
            conf['snmp_version'] = '2c'
            conf['snmp_community'] = opts.community
        else:
            if opts.cfg is None:
                print 'ERROR - No configuration file is specified'
                optp.print_help()
                sys.exit(1)
        if opts.hardware is None:
            check_all = True
            opts.no_alert = True
        elif '#' not in opts.hardware:
            check_all = False
            opts.no_alert = True
        else:
            check_all = False
            if opts.cfg is None:
                print 'ERROR - No configuration file is specified'
                sys.exit(1)
        if opts.mib is None:
            opts.mib = 'ALL'
        else:
            if os.path.isfile(opts.mib):
                try:
                    f = open(opts.mib, 'r')
                except Exception, error:
                    print 'ERROR - %s' % error
                    sys.exit(1)
                else:
                    f.close()
            else:
                print '%s not found' % opts.mib
                sys.exit(1)
        if opts.no_alert is True:
            pass
        else:
            if opts.ok:
                conf['ok'] = opts.ok
            else:
                if opts.cfg is None:
                    print 'OK state not set, and also configuration file not specified!'
                    sys.exit(1)
            if opts.warn:
                conf['warn'] = opts.warn
            else:
                if opts.cfg is None:
                    print 'WARNING state not set, and also configuration file not specified!'
                    sys.exit(1)
            if opts.crit:
                conf['crit'] = opts.crit
            else:
                if opts.cfg is None:
                    print 'CRITICAL state not set, and also configuration file not specified!'
                    sys.exit(1)
            if re.search(r'FAN', opts.hardware, re.IGNORECASE):
                if opts.fan_alert:
                    try:
                        conf['fan_min_warn'] = int(opts.fan_alert.split(',')[0])
                        conf['fan_min_crit'] = int(opts.fan_alert.split(',')[1])
                    except Exception:
                        print 'Error parsing threshold.'
                        sys.exit(1)
            elif re.search(r'VDISK', opts.hardware, re.IGNORECASE):
                if opts.vdiskbad:
                    try:
                        conf['vdisk_bad_warn'] = int(opts.vdiskbad.split(',')[0])
                        conf['vdisk_bad_crit'] = int(opts.vdiskbad.split(',')[1])
                    except Exception:
                        print 'Error parsing threshold.'
                        sys.exit(1)
            elif re.search(r'SENSOR', opts.hardware, re.IGNORECASE):
                if opts.temp_warn or opts.temp_crit:
                    try:
                        conf['sensor_temp_min_warn'] = float(opts.temp_warn.split(',')[0])
                        conf['sensor_temp_max_warn'] = float(opts.temp_warn.split(',')[1])
                        conf['sensor_temp_min_crit'] = float(opts.temp_crit.split(',')[0])
                        conf['sensor_temp_max_crit'] = float(opts.temp_crit.split(',')[1])
                    except Exception:
                        print 'Error parsing threshold.'
                        sys.exit(1)
            elif re.search(r'PS', opts.hardware, re.IGNORECASE):
                if opts.volt_warn or opts.volt_crit:
                    try:
                        conf['ps_vol_min_warn'] = float(opts.volt_warn.split(',')[0])
                        conf['ps_vol_max_warn'] = float(opts.volt_warn.split(',')[1])
                        conf['ps_vol_min_crit'] = float(opts.volt_crit.split(',')[0])
                        conf['ps_vol_max_crit'] = float(opts.volt_crit.split(',')[1])
                    except Exception:
                        print 'Error parsing threshold.'
                        sys.exit(1)
                if opts.cur_warn or opts.cur_crit:
                    try:
                        conf['ps_cur_min_warn'] = float(opts.cur_warn.split(',')[0])
                        conf['ps_cur_max_warn'] = float(opts.cur_warn.split(',')[1])
                        conf['ps_cur_min_crit'] = float(opts.cur_crit.split(',')[0])
                        conf['ps_cur_max_crit'] = float(opts.cur_crit.split(',')[1])
                    except Exception:
                        print 'Error parsing threshold.'
                        sys.exit(1)
                if opts.wat_warn or opts.wat_crit:
                    try:
                        conf['ps_watt_min_warn'] = float(opts.wat_warn.split(',')[0])
                        conf['ps_watt_max_warn'] = float(opts.wat_warn.split(',')[1])
                        conf['ps_watt_min_crit'] = float(opts.wat_crit.split(',')[0])
                        conf['ps_watt_max_crit'] = float(opts.wat_crit.split(',')[1])
                    except Exception:
                        print 'Error parsing threshold.'
                        sys.exit(1)
            elif re.search(r'PU', opts.hardware, re.IGNORECASE):
                if opts.consume:
                    try:
                        conf['pwr_consume_warn'] = float(opts.consume.split(',')[0])
                        conf['pwr_consume_crit'] = float(opts.consume.split(',')[1])
                    except Exception:
                        print 'Error parsing threshold.'
                        sys.exit(1)
        return check_all, opts.host, opts.hardware, opts.mib, opts.no_alert, opts.cfg, opts.perf


#----------------------CONFIG READER----------------------#
# This funciton read snmp and state alert definition in config file and also do last check of alert threshold.
def config_check(cfg_path, no_alert):
    try:
        cfg_f = open(cfg_path, 'r')
    except Exception, error:
        if conf['snmp_version'] == '2c' and no_alert is True:
            pass
        else:
            print error
            sys.exit(1)
    else:
        cfg_parameters = cfg_f.read().split('\n')
        cfg_f.close()
        cfg_cmt = re.compile('^#')
        #--parse config to config dict
        for parameter in cfg_parameters:
            if cfg_cmt.search(parameter) or parameter == '':
                pass
            else:
                par = parameter.split('=')
                if par[0].lower().strip() == 'snmp_version':
                    if conf['snmp_version'] == '2c':
                        continue
                if par[0].lower().strip() == 'ok' or par[0].lower().strip() == 'warn' or par[0].lower().strip() == 'crit':
                    if conf[par[0].lower().strip()] is None:
                        pass
                    else:
                        continue
                if par[1] == '':
                    par[1] = None
                try:
                    conf[par[0].lower().strip()] = par[1].strip()
                except KeyError:
                    print 'ERROR - Invalid config parameter:', par[0]
                    sys.exit(1)
                except AttributeError:
                    pass
    #--verify SNMP
    if conf['snmp_version'] is None:
        print 'ERROR - No SNMP version is specified'
        sys.exit(1)
    elif conf['snmp_version'] == '2c':
        if conf['snmp_community'] is None:
            print 'ERROR - Missing SNMPv2 community string'
            sys.exit(1)
    elif conf['snmp_version'] == '3':
        v3_cfg = [c for c in conf.keys() if 'snmp_v3' in c]
        tmp = [t for t in v3_cfg if conf[t] is None]
        if tmp:
            print 'ERROR - Missing parameters:\n' + '\n\t'.join(tmp)
            sys.exit(1)
        else:
            sec_lvl = re.compile('noAuthNoPriv|authNoPriv|authPriv', re.IGNORECASE)
            authen = re.compile('sha|md5', re.IGNORECASE)
            privacy = re.compile('des|aes', re.IGNORECASE)
            if not any(sec_lvl.search(conf[cfg]) for cfg in v3_cfg):
                print 'ERROR - Security level should be: noAuthNoPriv, authNoPriv or authPriv'
                sys.exit(1)
            if not any(authen.search(conf[cfg]) for cfg in v3_cfg):
                print 'ERROR - Authentication protocol should be: MD5 or SHA'
                sys.exit(1)
            if not any(privacy.search(conf[cfg]) for cfg in v3_cfg):
                print 'ERROR - Privacy protocol should be: DES or AES'
                sys.exit(1)
    #--verify alert
    if no_alert is True:
        pass
    else:
        #--verify threshold & set default if None
        if conf['fan_min_warn'] is None:
            conf['fan_min_warn'] = 840
        if conf['fan_min_crit'] is None:
            conf['fan_min_crit'] = 600
        if conf['pwr_consume_warn'] is None:
            conf['pwr_consume_warn'] = 896
        if conf['pwr_consume_crit'] is None:
            conf['pwr_consume_crit'] = 980
        if conf['vdisk_bad_warn'] is None:
            conf['vdisk_bad_warn'] = 5
        if conf['vdisk_bad_crit'] is None:
            conf['vdisk_bad_crit'] = 10
        if conf['sensor_temp_min_warn'] is None:
            conf['sensor_temp_min_warn'] = [3, 8, 8, 8]
        if conf['sensor_temp_min_crit'] is None:
            conf['sensor_temp_min_crit'] = [-7, 3, 3, 3]
        if conf['sensor_temp_max_warn'] is None:
            conf['sensor_temp_max_warn'] = [42, 70, 85, 85]
        if conf['sensor_temp_max_crit'] is None:
            conf['sensor_temp_max_crit'] = [47, 75, 90, 90]
        #--verify state alert
        if conf['crit'] is None or conf['warn'] is None or conf['ok'] is None:
            print 'ERROR - Alert definitions not set. See -h or -m.'
            sys.exit(1)
        else:
            #--detect overlap state alert
            conf['ok'] = conf['ok'].split('|')
            conf['warn'] = conf['warn'].split('|')
            conf['crit'] = conf['crit'].split('|')
            for w in conf['warn']:
                if w in conf['ok'] or w in conf['crit']:
                    print 'ERROR - State alert overlapped!'
                    sys.exit(1)
            for c in conf['crit']:
                if c in conf['ok']:
                    print 'ERROR - State alert overlapped!'
                    sys.exit(1)
            #--rejoin state alert
            conf['ok'] = '|'.join(conf['ok'])
            conf['warn'] = '|'.join(conf['warn'])
            conf['crit'] = '|'.join(conf['crit'])
    return no_alert


#----------------------PARSER----------------------#
class PARSER:
    def __init__(self, host, hardware, order, alert, mib, perf):
        self.hardware = hardware
        self.order = order
        self.no_alert = alert
        self.mib = mib
        self.host = host
        self.perf = perf
        self.snmp_type = ''

    def get_snmp(self, oids):
        v3_cmd = '%s %s -O q -v %s -u %s -l %s -a %s -A %s -x %s -X %s %s -m %s'\
                 % (self.snmp_type, self.host, conf['snmp_version'], conf['snmp_v3_user'],
                    conf['snmp_v3_security_level'],
                    conf['snmp_v3_authentication_protocol'], conf['snmp_v3_authentication_password'],
                    conf['snmp_v3_privacy_protocol'], conf['snmp_v3_privacy_password'],
                    oids, self.mib)  # command to use with snmp v3
        v2_cmd = '%s %s -O q -v %s -c %s %s -m %s' \
                 % (self.snmp_type, self.host, conf['snmp_version'],
                    conf['snmp_community'],
                    oids, self.mib)  # command to use with snmp v2
        cmd_list = {'3': v3_cmd,
                    '2': v2_cmd,
                    '2c': v2_cmd}
        try:
            snmp_cmd = cmd_list[conf['snmp_version']]
        except KeyError:
            print 'SNMP version %s not supported' % conf['snmp_version']
            sys.exit(1)
        #--start getting snmp data
        _, output = run(snmp_cmd)
        if _ != 0:
            if 'Unknown Object Identifier' in output:
                print 'Your MIB may out of dated!'
                print 'ERROR -', output.replace('\n', '. ')
            elif 'Timeout:' in output:
                print 'SNMP Timeout!'
            else:
                print output
            sys.exit(1)
        else:
            #--debug print output
            if 'No Such Instance currently exists at this OID' in output:
                print 'Hardware not found. If you sure the hw exists then you may want to edit TRANSLATOR code (line 612).'
                sys.exit(1)
            #--do extra queries for VOLT & CURRENT
            if self.order is None:
                if self.hardware[2] == 'PS':
                    _, __ = run(snmp_cmd.replace('powerSupplyTable', self.hardware[1][0]))
                    output += '\n' + __
                    _, __ = run(snmp_cmd.replace('powerSupplyTable', self.hardware[1][1]))
                    output += '\n' + __
                elif self.hardware[2] == 'PU':
                    _, __ = run(snmp_cmd.replace('powerUnitTable', self.hardware[1][0]))
                    output += '\n' + __
                    _, __ = run(snmp_cmd.replace('powerUnitTable', self.hardware[1][1]))
                    output += '\n' + __
            return output.split('\n')

    def classifier(self, data, tmp):  # classify snmp data to it's specific type
        item = re.compile(self.hardware[3])
        for _ in data:
            if item.search(_):
                #--debug print 'matched:', _
                item_order = int(_.split()[0].split('.')[-1])
                item_info = ' '.join(_.split()[1:])
                if self.hardware[2] == 'PS':
                    if 'voltageProbeReading' in _:
                        item_order -= 25  # ps volt starting with number 26
                elif self.hardware[2] == 'PU':
                    if 'Current' in _:
                        continue
                    elif 'System Board Pwr Consumption' in _:  # get pwr consumption
                        for x in reversed(data):
                            if re.search(r'amperageProbeReading.1.%d' %item_order, x):
                                item_info = ' '.join(x.split()[1:])
                                item_order = 1
                                break
                try:
                    tmp[item_order].append(item_info)
                except KeyError:
                    continue
        # dump blank data to non-exists item, make it compatible with lower idrac 7 firmware version
        for t in tmp:
            if len(self.hardware[3].split('|')) > len(tmp[t]):
                for r in range(len(tmp[t]), len(self.hardware[3].split('|'))):
                    tmp[t].append('(n/a)')
        return tmp

    def raise_alert(self, tmp, value_on_alert):
        code = {0: [0, 'OK'],
                1: [1, 'WARN'],
                2: [2, 'CRIT'],
                3: [3, 'UNKNOWN']}
        alert = 0
        for key in tmp.keys():
            for stat in value_on_alert:  # check every status of hw
                if tmp[key][int(stat)] == '(n/a)':
                    continue
                if type(stat) == int:
                    if conf['warn'] == '$ALL$':
                        #debug print 'WARN ALL'
                        if re.search(conf['ok'], tmp[key][stat], re.IGNORECASE):
                            continue
                        elif re.search(conf['crit'], tmp[key][stat], re.IGNORECASE):
                            tmp[key][stat] += '(!!)'  # more useful with check_mk frontend
                            alert = 2
                        else:
                            tmp[key][stat] += '(!)'
                            if alert != 2: alert = 1
                    elif conf['crit'] == '$ALL$':
                        #debug print 'CRIT ALL'
                        if re.search(conf['ok'], tmp[key][stat], re.IGNORECASE):
                            continue
                        elif re.search(conf['warn'], tmp[key][stat], re.IGNORECASE):
                            tmp[key][stat] += '(!)'
                            if alert != 2: alert = 1
                        else:
                            tmp[key][stat] += '(!!)'
                            alert = 2
                    else:
                        #debug print 'NO ALL'
                        if re.search(conf['ok'], tmp[key][stat], re.IGNORECASE):
                            continue
                        elif re.search(conf['warn'], tmp[key][stat], re.IGNORECASE):
                            tmp[key][stat] += '(!)'
                            if alert != 2: alert = 1
                        elif re.search(conf['crit'], tmp[key][stat], re.IGNORECASE):
                            tmp[key][stat] += '(!!)'
                            alert = 2
                        else:
                            tmp[key][stat] += '(?)'
                            if alert != 2 and alert != 1: alert = 3
                else:
                    stat_t = int(stat)
                    if self.hardware[2] == 'Fan':
                        if int(tmp[key][stat_t]) <= conf['fan_min_crit']:
                            tmp[key][stat_t] += '(!!)'
                            alert = 2
                        elif int(tmp[key][stat_t]) <= conf['fan_min_warn']:
                            tmp[key][stat_t] += '(!)'
                            if alert != 2:
                                alert =1
                    elif self.hardware[2] == 'PS':
                        if stat_t == 6:
                            tmp[key][stat_t] = float(tmp[key][stat_t])/1000
                            if tmp[key][stat_t] >= conf['ps_vol_max_crit'] or tmp[key][stat_t] <= conf['ps_vol_min_crit']:
                                tmp[key][stat_t] = '%.0f(!!)' %tmp[key][stat_t]
                                alert = 2
                            elif tmp[key][stat_t] >= conf['ps_vol_max_warn'] or tmp[key][stat_t] <= conf['ps_vol_min_warn']:
                                tmp[key][stat_t] = '%.0f(!)' %tmp[key][stat_t]
                                if alert != 2:
                                    alert = 1
                            else: tmp[key][stat_t] = '%.0f' %tmp[key][stat_t]
                        elif stat_t == 5:
                            tmp[key][stat_t] = float(tmp[key][stat_t])/10
                            if tmp[key][stat_t] >= conf['ps_cur_max_crit'] or tmp[key][stat_t] <= conf['ps_cur_min_crit']:
                                tmp[key][stat_t] = '%.1f(!!)' %tmp[key][stat_t]
                                alert = 2
                            elif tmp[key][stat_t] >= conf['ps_cur_max_warn'] or tmp[key][stat_t] <= conf['ps_cur_min_warn']:
                                tmp[key][stat_t] = '%.1f(!)' %tmp[key][stat_t]
                                if alert != 2:
                                    alert = 1
                            else: tmp[key][stat_t] = '%.1f' %tmp[key][stat_t]
                        elif stat_t == 2:
                            tmp[key][stat_t] = float(tmp[key][stat_t])/10
                            if tmp[key][stat_t] >= conf['ps_watt_max_crit'] or tmp[key][stat_t] <= conf['ps_watt_min_crit']:
                                tmp[key][stat_t] = '%.0f(!!)' %tmp[key][stat_t]
                                alert = 2
                            elif tmp[key][stat_t] >= conf['ps_watt_max_warn'] or tmp[key][stat_t] <= conf['ps_watt_min_warn']:
                                tmp[key][stat_t] = '%.0f(!)' %tmp[key][stat_t]
                                if alert != 2:
                                    alert = 1
                            else: tmp[key][stat_t] = '%.0f' %tmp[key][stat_t]
                    elif self.hardware[2] == 'PU':
                        if float(tmp[key][stat_t]) >= conf['pwr_consume_crit']:
                            tmp[key][stat_t] += '(!!)'
                            alert = 2
                        elif float(tmp[key][stat_t]) >= conf['pwr_consume_warn']:
                            tmp[key][stat_t] += '(!)'
                            if alert != 2:
                                alert = 1
                    elif self.hardware[2] == 'Sensor':
                        tmp[key][stat_t] = float(tmp[key][stat_t])/10
                        if type(conf['sensor_temp_min_warn']) != list\
                            or type(conf['sensor_temp_max_warn']) != list\
                            or type(conf['sensor_temp_min_crit']) != list\
                            or type(conf['sensor_temp_max_crit']) != list:
                            min_warn = conf['sensor_temp_min_warn']
                            max_warn = conf['sensor_temp_max_warn']
                            min_crit = conf['sensor_temp_min_crit']
                            max_crit = conf['sensor_temp_max_crit']  # this is when you forgot to set threshold
                        else:
                            if 'System Board Inlet Temp' in tmp[key][-1]:
                                max_crit = conf['sensor_temp_max_crit'][0]
                                min_crit = conf['sensor_temp_min_crit'][0]
                                max_warn = conf['sensor_temp_max_warn'][0]
                                min_warn = conf['sensor_temp_min_warn'][0]
                            elif 'System Board Exhaust Temp' in tmp[key][-1]:
                                max_crit = conf['sensor_temp_max_crit'][1]
                                min_crit = conf['sensor_temp_min_crit'][1]
                                max_warn = conf['sensor_temp_max_warn'][1]
                                min_warn = conf['sensor_temp_min_warn'][1]
                            elif 'CPU' in tmp[key][-1]:
                                cpu_number = int(tmp[key][4].split()[0].split('CPU')[1])  # in case you have more than 1 CPU
                                max_crit = conf['sensor_temp_max_crit'][cpu_number+1]
                                min_crit = conf['sensor_temp_min_crit'][cpu_number+1]
                                max_warn = conf['sensor_temp_max_warn'][cpu_number+1]
                                min_warn = conf['sensor_temp_min_warn'][cpu_number+1]
                        if tmp[key][stat_t] >= max_crit or tmp[key][stat_t] <= min_crit:
                            tmp[key][stat_t] = '%.1f(!!)' %tmp[key][stat_t]
                            alert = 2
                        elif tmp[key][stat_t] >= max_warn or tmp[key][stat_t] <= min_warn:
                            tmp[key][stat_t] = '%.1f(!)' %tmp[key][stat_t]
                            if alert != 2: alert = 1
                        else: tmp[key][stat_t] = '%.1f' %tmp[key][stat_t]
        return tmp, code[alert]  # return with exit code

    def main(self):
        hw_dict = {}
        hw_quantity = 0
        if self.order is None:
            self.snmp_type = 'snmpwalk'  # use walk for non specified hw order
            snmp_data = self.get_snmp(oids=self.hardware[0])
            for data in snmp_data:
                if re.search(self.hardware[3].split('|')[0], data, re.IGNORECASE):
                    hw_quantity += 1
            for _ in range(0, hw_quantity):
                hw_dict[_+1] = []  # prepare slot for number of hw
        else:
            hw_dict[self.order] = []
            self.snmp_type = 'snmpget'
            # DISK/VDISK use suffix .1 for first disk
            # Others use .1.1 for first device
            # If your idrac installed with more hw, write your own translator here
            #--TRANSLATOR--#
            if self.hardware[2] == 'PDisk' or self.hardware[2] == 'VDisk':
                oids = self.hardware[3].replace('|', ' ').replace('\.', '.%d' % self.order)
                snmp_data = self.get_snmp(oids)
            else:
                oids = self.hardware[3].replace('|', ' ').replace('\.', '.1.%d' % self.order)
                if self.hardware[2] == 'PS':
                    oids = ' '.join(oids.split()[:-1])
                    snmp_data = self.get_snmp(oids)
                    self.snmp_type = 'snmpwalk'
                    for oid in self.hardware[1]:
                        oids = oid
                        snmp_data = snmp_data + self.get_snmp(oids)
                elif self.hardware[2] == 'PU':
                    oids = ' '.join(oids.split()[:-2])
                    snmp_data = self.get_snmp(oids)
                    self.snmp_type = 'snmpwalk'
                    for oid in self.hardware[1]:
                        oids = oid
                        snmp_data = snmp_data + self.get_snmp(oids)
                else:
                    snmp_data = self.get_snmp(oids)
            #--END OF TRANSLATOR--#
        #--debug print '\n'.join(snmp_data)
        #--debug print hw_dict
        hw_dict = self.classifier(snmp_data, hw_dict)  # classify data
        #--debug print hw_dict
        #--re-format output to suit hw type. Power is the most messed part!
        output = []
        exit_code = [0, 'OK']
        if self.hardware[2] == 'PDisk':
            value_on_alert = [3, 7]  # raise alert on these value. Mapped with all_hardware as list order. int type for status check, string type for range check
            if self.no_alert is False:
                hw_dict, exit_code = self.raise_alert(hw_dict, value_on_alert)
            tmp = '%s %s (%s) %s GB: %s, PowerStat: %s, HotSpare: %s [%s, S/N: %s]'
            for hw in hw_dict.values():
                for v in value_on_alert:
                    v = int(v)
                    hw[v] = hw[v].upper()
                if hw[5] == '(N/A)':
                    hw_5 = hw[5]
                else: hw_5 = round(float(hw[5])/1024, 2)
                output.append(tmp % (self.hardware[2], hw[0], hw[1].split()[-1].replace('"', ''), hw_5, hw[3], hw[7], hw[6].replace('HotSpare', '').replace('notASpare', 'no'), hw[2].replace('"', ''), hw[4].replace('"', '')))
        elif self.hardware[2] == 'Fan':
            value_on_alert = [1, 2, '3']
            if self.no_alert is False:
                hw_dict, exit_code = self.raise_alert(hw_dict, value_on_alert)
            tmp = '%s: %s RPM - %s/%s%s'
            for hw in hw_dict.values():
                for v in value_on_alert:
                    v = int(v)
                    hw[v] = hw[v].upper()
                if self.perf is True:
                    perf_data = ' | RPM=%s;%s;%s;;' % (hw[3].split('(')[0], conf['fan_min_warn'], conf['fan_min_crit'])
                else:
                    perf_data = ''
                output.append(tmp % (hw[4].replace('"', '').replace(' RPM', ''), hw[3], hw[1], hw[2], perf_data))
        elif self.hardware[2] == 'Sensor':
            value_on_alert = [1, 2, '3']  # int type for status check, str type for range check
            if self.no_alert is False:
                hw_dict, exit_code = self.raise_alert(hw_dict, value_on_alert)
                tmp = '%s: %s C %s/%s%s'
            else:
                tmp = '%s: %s C %s/%s%s'
            for hw in hw_dict.values():
                for v in value_on_alert:
                    v = int(v)
                    hw[v] = hw[v].upper()
                if self.perf is True:
                    if type(conf['sensor_temp_max_crit']) == list or type(conf['sensor_temp_max_warn']) == list:
                        perf_data = ' | Temperature=%sC;;;;' % (hw[3])
                    else:
                        perf_data = ' | Temperature=%sC;%s;%s;%s;%s' % (hw[3], conf['sensor_temp_min_warn'], conf['sensor_temp_max_warn'], conf['sensor_temp_min_crit'], conf['sensor_temp_max_crit'])
                else:
                    perf_data = ''
                if self.no_alert is False:
                    output.append(tmp % (hw[4].replace('"', ''), hw[3], hw[1], hw[2], perf_data))
                else:
                    if hw[3] == '(N/A)':
                        hw_3 = hw[3]
                    else: hw_3 = round(float(hw[3])/10, 1)
                    output.append(tmp % (hw[4].replace('"', ''), hw_3, hw[1], hw[2], perf_data))
        elif self.hardware[2] == 'CPU':
            value_on_alert = [1, 2]
            if self.no_alert is False:
                hw_dict, exit_code = self.raise_alert(hw_dict, value_on_alert)
            tmp = '%s %s (%s cores/%s threads): %s/%s [%s]'
            for hw in hw_dict.values():
                for v in value_on_alert:
                    v = int(v)
                    hw[v] = hw[v].upper()
                output.append(tmp % (self.hardware[2], hw[0], hw[3].split()[-1], hw[4].split()[-1], hw[1], hw[2], hw[5].replace('"', '')))
        elif self.hardware[2] == 'PU':
            value_on_alert = [1, 2, 3, '4']
            if self.no_alert is False:
                hw_dict, exit_code = self.raise_alert(hw_dict, value_on_alert)
            tmp = '%s %s: %s/%s, RedundancyStatus: %s, SystemBoard Pwr Consumption: %s W%s'
            for hw in hw_dict.values():
                if len(hw) < 5:  # just in case no pwr consumption found
                    hw.append('(N/A)')
                for v in value_on_alert:
                    v = int(v)
                    hw[v] = hw[v].upper()
                if self.perf is True:
                    perf_data = ' | PwrConsumption=%sW;%s;%s;;' % (hw[4].split('(')[0], conf['pwr_consume_warn'], conf['pwr_consume_crit'])
                else:
                    perf_data = ''
                output.append(tmp % (self.hardware[2], hw[0], hw[1], hw[3], hw[2], hw[4], perf_data))
        elif self.hardware[2] == 'PS':
            value_on_alert = [1, '2', '6', '5']
            if self.no_alert is False:
                hw_dict, exit_code = self.raise_alert(hw_dict, value_on_alert)
                tmp = '%s %s: %s, Volt I/O: %s V/%s V, Current: %s A, Watt I/O: %s W/%s W%s'
            else:
                tmp = '%s %s: %s, Volt I/O: %s V/%s V, Current: %s A, Watt I/O: %s W/%s W'
            for hw in hw_dict.values():
                for v in value_on_alert:
                    v = int(v)
                    hw[v] = hw[v].upper()
                if self.perf is True:
                    if hw[4] == '(N/A)':
                        hw_4 = hw[4]
                    else: hw_4 = round(float(hw[4])/10, 0)
                    perf_data = ' | VoltINPUT=%sV;;;; VoltOUTPUT=%sV;;;; Current=%sA;;;; WattINPUT=%sW;;;; WattOUTPUT=%sW;;;;' %(hw[3], hw[6].split('(')[0], hw[5].split('(')[0], hw_4, hw[2].split('(')[0])
                else:
                    perf_data = ''
                if self.no_alert is False:
                    if hw[4] == '(N/A)':
                        hw_4 = hw[4]
                    else: hw_4 = float(hw[4])/10
                    output.append(tmp % (self.hardware[2], hw[0], hw[1], hw[3], hw[6], hw[5], hw_4, hw[2], perf_data))
                else:
                    if hw[4] == '(N/A)':
                        hw_4 = hw[4]
                    else: hw_4 = float(hw[4])/10
                    if hw[6] == '(N/A)':
                        hw_6 = hw[6]
                    else: hw_6 = float(hw[6])/1000
                    if hw[5] == '(N/A)':
                        hw_5 = hw[5]
                    else: hw_5 = float(hw[5])/10
                    if hw[2] == '(N/A)':
                        hw_2 = hw[2]
                    else: hw_2 = float(hw[2])/10
                    output.append(tmp % (self.hardware[2], hw[0], hw[1], hw[3], hw_6, hw_5, hw_4, hw_2))
        elif self.hardware[2] == 'Memory':
            value_on_alert = [1, 2]
            if self.no_alert is False:
                hw_dict, exit_code = self.raise_alert(hw_dict, value_on_alert)
            #--debug print hw_dict
            tmp = '%s %s (%s) %s GB/%s MHz: %s/%s [%s, %s, S/N: %s]'
            for hw in hw_dict.values():
                for v in value_on_alert:
                    v = int(v)
                    hw[v] = hw[v].upper()
                if hw[5] == '(N/A)':
                    hw_5 = hw[5]
                else: hw_5 = round(float(hw[5].split()[-1])/(1024*1024), 2)
                output.append(tmp % (self.hardware[2], hw[0], hw[4].replace('.', ' ').replace('"', ''), hw_5, hw[6].split()[-1], hw[1], hw[2], hw[3].replace('deviceTypeIs', ''), hw[7].replace('"', ''), hw[8].replace('"', ''),))
        elif self.hardware[2] == 'Battery':
            value_on_alert = [1, 2, 3]
            if self.no_alert is False:
                hw_dict, exit_code = self.raise_alert(hw_dict, value_on_alert)
            tmp = '%s: %s/%s [%s]'
            for hw in hw_dict.values():
                for v in value_on_alert:
                    v = int(v)
                    hw[v] = hw[v].upper()
                output.append(tmp % (hw[4].replace('"', ''), hw[1], hw[2], hw[3]))
        elif self.hardware[2] == 'VDisk':
            value_on_alert = [2, 5, '6']
            if self.no_alert is False:
                hw_dict, exit_code = self.raise_alert(hw_dict, value_on_alert)
            tmp = '%s %s (%s): %s/%s, %s (%s GB), BadBlock: %s [%s]'
            for hw in hw_dict.values():
                for v in value_on_alert:
                    v = int(v)
                    hw[v] = hw[v].upper()
                if hw[3] == '(N/A)':
                    hw_3 = hw[3]
                else: hw_3 = round(float(hw[3])/1024, 2)
                output.append(tmp % (self.hardware[2], hw[0], hw[1].replace('"', ''), hw[5], hw[2], hw[4].replace('r', 'RAID-'), hw_3, hw[6], hw[7].replace('"', '')))
        return output, exit_code


#----------------------MANUAL----------------------#
def manual():
    try:
        f = open('README', 'r')
    except IOError:
        print """README not found <(' .' )>"""
    else:
        print ''.join(f.readlines())
        f.close()


#----------------------CONFIG GENERATE----------------------#
def generate_config():
    configuration = \
"""#SNMP CONFIGURATIONS
SNMP_version = 3
#SNMP_community = 'public'
SNMP_v3_user = 'my user'
#SNMP_v3_security_level = noAuthNoPriv|authNoPriv|authPriv
SNMP_v3_security_level = authPriv
#SNMP_v3_authentication_protocol = sha|md5
SNMP_v3_authentication_protocol = sha
SNMP_v3_authentication_password = 'mypassword'
#SNMP_v3_privacy_protocol = des|aes
SNMP_v3_privacy_protocol = aes
SNMP_v3_privacy_password = 'mypassword'

#STATE ALERT
#You can use state alert with option --ok/--warn/--crit as well.
#User can defines state alert based on device status
#Conditions are separated by "|"
#$ALL$ means all status that not belong to others.
#OK = $ALL$ means --no-alert
OK = ok|online|spunup|full|ready|enabled|presence
WARN = $ALL$
CRIT = critical|nonRecoverable|fail
"""
    try:
        f = open('check_idrac.conf', 'w')
        f.write(configuration)
    except Exception, error:
        print error
        sys.exit(1)
    else:
        f.close()
    print 'Configuration generated as check_idrac.conf'

#----------------------MAIN----------------------#
if __name__ == '__main__':
    #INIT
    name = None
    hw_order = None
    #LOAD CONFIG
    check_all, host, hw_name, hw_mib, hw_no_alert, hw_cfg, perf = option_reader()  # from cmd
    if hw_no_alert is True:
        perf = None
    #START CHECKING
    #--debug print check_all
    if check_all is True:  # check all hw
        hw_no_alert = config_check(hw_cfg, hw_no_alert)
        for hw_name in all_hardware.keys():
            hw_info = all_hardware[hw_name]
            result, tmp_code = PARSER(host, hw_info, hw_order, hw_no_alert, hw_mib, perf).main()
            print '%s' % hw_name
            sys.stdout.write('--')
            print '\n--'.join(result)
        sys.exit(0)
    else:  # check for specific hw
        not_found = 0
        for _ in all_hardware.keys():
            verify = re.compile(r'^%s$' % _, re.IGNORECASE)
            verify_sub = re.compile(r'^%s#' % _, re.IGNORECASE)
            if verify.search(hw_name):
                name = hw_name
                #--debug print name
                break
            elif verify_sub.search(hw_name):
                hw_no_alert = False
                name = hw_name.split('#')[0]
                try:
                    hw_order = int(hw_name.split('#')[1])
                except ValueError, error:
                    print 'ERROR - Hardware order must be INTEGER'
                    sys.exit(1)
                else:
                    if hw_order <= 0:
                        print 'ERROR - Hardware order must be positive'
                        sys.exit(1)
                    else:
                        break
            else:
                not_found += 1
        if not_found == len(all_hardware.keys()):
            print 'SORRY - Hardware not found! --help to see available hardware.'
            sys.exit(1)
        else:
            hw_info = all_hardware[name.upper()]  # get hw info from hw dict
            #--debug print name, hw_order
            if hw_order or conf['snmp_version'] is None:
                hw_no_alert = config_check(hw_cfg, hw_no_alert)  # from file
            result, exit_code = PARSER(host, hw_info, hw_order, hw_no_alert, hw_mib, perf).main()
            if hw_order is None:  # for not specify hw part
                print '\n'.join(result)
            else:
                print '%s - %s' %(exit_code[1], '\n'.join(result))
            sys.exit(exit_code[0])
