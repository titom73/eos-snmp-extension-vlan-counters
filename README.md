# EOS SNMP Extension

## About

This script provides an example of how to leverage snmp-server extension on Arista EOS devices to create a custom set of OIDs to expose counters from `show vlan counters`

It leverages [__pass_persist module__](https://github.com/nagius/snmp_passpersist) to create a new SNMP structure available for your monitoring system.

SNMP structure generated by script use following template:

```
{{ OID_BASE }}.{{ VLAN_ID }}.{{ OID_DESCRIPTION | OID_VALUE }}.{{ OID_TRANSLATION[counter] }}
```

## Getting started

1. Copy this script to /mnt/flash as vlanCounters.py

```shell
admin@eos# bash
$ cd /mnt/flash
$ vi /mnt/flash/vlanCounters.py
$ sudo chmod +x vlanCounters.py
```

2. Copy [snmp_passpersist](https://github.com/nagius/snmp_passpersist) to `/mnt/flash`

```shell
admin@eos# bash
$ cd /mnt/flash
$ wget https://raw.githubusercontent.com/nagius/snmp_passpersist/master/snmp_passpersist.py
```

3. Enable management api (script uses a unix socket)

```eos
management api http-commands
   protocol unix-socket
   no shutdown
```

4. Enable Hardware counters for vlan

```eos
hardware counter feature vlan out
hardware counter feature vlan in
```

5. Configure SNMP-SERVER extension

```eos
snmp-server extension .1.3.6.1.3.53.9 flash:/vlanCounters.py
```

6. Re-configure SNMP-SERVER extension when script is updated

```eos
no snmp-server extension .1.3.6.1.3.53.9 flash:/vlanCounters.py
snmp-server extension .1.3.6.1.3.53.9 flash:/vlanCounters.py
```

> SNMP service __MUST__ be configured on your EOS device with at least one community.

## Get information from SNMP

From your EOS device:

```eos
s7289(config)#show snmp mib walk .1.3.6.1.3.53.9

SNMPv2-SMI::experimental.53.9.0 = STRING: "MIB updated by vlanCounters.py"
SNMPv2-SMI::experimental.53.9.0.1.0.1 = STRING: "inUcastPkts"
SNMPv2-SMI::experimental.53.9.0.1.0.2 = STRING: "outUcastPkts"
SNMPv2-SMI::experimental.53.9.0.1.0.3 = STRING: "inOctets"
SNMPv2-SMI::experimental.53.9.0.1.0.4 = STRING: "outOctets"
SNMPv2-SMI::experimental.53.9.0.1.0.5 = STRING: "inBcastPkts"
SNMPv2-SMI::experimental.53.9.0.1.0.6 = STRING: "outMcastPkts"
SNMPv2-SMI::experimental.53.9.0.1.1.1 = INTEGER: 77934
SNMPv2-SMI::experimental.53.9.0.1.1.2 = INTEGER: 0
SNMPv2-SMI::experimental.53.9.0.1.1.3 = INTEGER: 12207903
SNMPv2-SMI::experimental.53.9.0.1.1.4 = INTEGER: 0
SNMPv2-SMI::experimental.53.9.0.1.1.5 = INTEGER: 0
SNMPv2-SMI::experimental.53.9.0.1.1.6 = INTEGER: 0
```
