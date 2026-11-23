# 07 - Troubleshooting and Known Issues

## Contents

- Problems met during the labs
- Common faults in this design
- Diagnostic commands
- Known issues

## Problems met during the labs

| Problem | Cause | Resolution |
| --- | --- | --- |
| VLAN 1 could not be renamed | VLAN 1 is the reserved default VLAN on Cisco switches | Left as `default`. Planned name dropped |
| Core trunk would not work without extra configuration | The Catalyst 3560 supports both 802.1Q and ISL and must be told which to use | Added `switchport trunk encapsulation dot1q` before `switchport mode trunk`. Not needed on the 2960 |
| IP phone not working in Packet Tracer | The phone starts powered off in the simulator | Powered it on from the *Physical* tab by connecting the power adapter |
| “Bad Value” returned by some MIB Browser entries | Packet Tracer supports standard MIBs but not all Cisco-specific MIB extensions | No fix needed. Understood as a simulator limitation |
| System Contact blank in the MIB Browser | `snmp-server contact` was not configured | Expected result. The command is listed as optional |

## Common faults in this design

These are failure modes described in the documentation, with the check that finds them.

| Symptom | Likely cause | Check |
| --- | --- | --- |
| Devices on different switches in the same VLAN cannot communicate, with no obvious error | VLAN missing from the database of one switch in the path. The switch cannot forward its traffic | `show vlan` on every switch in the path |
| All devices in several VLANs lose connectivity after a trunk change | `switchport trunk allowed vlan <id>` was used without `add`, replacing the allowed list | `show interfaces trunk`, `show running-config`. Restore with `switchport trunk allowed vlan 1,2,3,4` |
| Untagged traffic lands in the wrong VLAN | Native VLAN differs between the two ends of a trunk | Compare the native VLAN on both ends with `show interfaces <port> switchport` |
| Trunk does not form on a 3560 | Encapsulation not set | Check for `switchport trunk encapsulation dot1q` |
| A port unexpectedly becomes a trunk | Port left on dynamic negotiation | Set `switchport mode access` explicitly on end-device ports |
| Voice not working on the phone port | Phone powered off (simulator), or no voice VLAN on the port | Check phone power. Check `switchport voice vlan 2` |
| SNMP queries get no response | Manager cannot reach the device, community string mismatch, or SNMP not configured | `ping` the agent from the manager. Compare the community string. `show snmp` |
| SNMP settings missing after a reboot | Configuration was not saved | `write memory` after configuring |

## Diagnostic commands

| Command | Use |
| --- | --- |
| `show vlan` | VLANs and port assignments on a switch |
| `show interfaces <port> switchport` | Port mode, trunking protocol, native VLAN |
| `show interfaces trunk` | Trunk ports, native VLAN and allowed VLANs |
| `show running-config` | Current configuration, to confirm settings and allowed VLANs |
| `show snmp` | SNMP status on a router or switch |
| `ping <address>` | Basic reachability between devices |

## Known issues

These are limitations or gaps in the labs as documented. They are not fixed in this repository.

| # | Issue | Impact |
| --- | --- | --- |
| 1 | **No inter-VLAN routing** configured. The Layer 3 core could provide it, but no routed interfaces are documented | Devices in different VLANs, such as a PC in VLAN 1 and the server in VLAN 4, would not be able to reach each other. Only same-VLAN connectivity (PC 1 to PC 4) was tested |
| 2 | **Default gateway 192.168.1.1** is set on PC 1 but no Lab 1 device with that address is documented | The gateway is not used by the same-VLAN test |
| 3 | **VLAN 1 used for management and user PCs**, and as the native VLAN | Chosen to keep the lab simple. Management traffic shares a VLAN with users |
| 4 | **Phone port differs between diagrams.** The Step 1.5 diagram shows the IP phone on Fa0/3 of Access Switch 02 with `switchport access vlan 2`. The written configuration in 1.6 uses Fa0/1 with `switchport access vlan 1` and `switchport voice vlan 2` | This repository uses the written 1.6 configuration |
| 5 | **PC count differs.** Five PCs in the IP table, four described in the port configuration | Placement of the fifth PC is not documented |
| 6 | **Access Switch 02 and 03 uplink ports not recorded** | Config files for these switches contain a comment, not an uplink configuration |
| 7 | **Router SNMP configuration not saved.** The router steps show `snmp-server community private RO` and no `write memory`. The switch steps include it | A router restart would remove the configuration |
| 8 | **STP not examined** | Failover behaviour of the dual-uplink design is assumed from default STP, not observed |
| 9 | **Packet Tracer MIB limits** | CPU and memory data are not available in the MIB Browser |
| 10 | **No Packet Tracer project file** in this repository | The labs cannot be opened directly. They have to be rebuilt from the configs and documentation |

Next: Security Considerations
