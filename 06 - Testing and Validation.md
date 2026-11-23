# 06 - Testing and Validation

Each configuration stage was checked before the next one started. This page lists the checks that are recorded in the lab documentation, what they showed, and what was not tested.

## Contents

- Lab 1 checks
- End-to-end connectivity test
- Lab 2 checks
- Not tested

## Lab 1 checks

| # | Check | Method | Result |
| --- | --- | --- | --- |
| 1 | VLAN database consistency | `show vlan` on all five switches | VLANs 1, 2, 3 and 4 present with the correct names on every switch |
| 2 | Reserved VLAN | Attempt to rename VLAN 1 | Rejected by IOS. VLAN 1 is reserved and keeps the name `default` |
| 3 | Core trunk status | `show interfaces fastethernet 0/1 switchport` on both core switches | Used to confirm trunk operation, native VLAN and trunking protocol before moving on |
| 4 | Core trunk configuration | `show running-config` on both core switches | Used to confirm trunk settings and allowed VLANs were present |
| 5 | Access uplink trunk status | `show interfaces trunk` on Access Switch 01 (listed in the Step 3 diagram) | Uplinks checked for trunk mode, native VLAN and allowed VLANs |
| 6 | Cross-switch connectivity | `ping 192.168.1.50` from PC 1 to PC 4 | Successful |

## End-to-end connectivity test

PC 1 and PC 4 are both in VLAN 1 but on different access switches. The ping from PC 1 to PC 4 had to travel from PC 1’s access switch, over a trunk to a core switch, across the core, and back down a trunk to the access switch serving PC 4:

```
PC> ping 192.168.1.50
```

It succeeded. A successful ping depends on every earlier step being correct: the VLANs on each switch, the access port assignments, the trunk links, the native VLAN, the allowed VLAN list and 802.1Q on the core. A fault in any of these would have caused it to fail.

<img width="1024" height="559" alt="image" src="https://github.com/user-attachments/assets/99a67694-0cae-4c56-82b5-d9e9ad60a93a" />
 PC 1 *Command Prompt* showing `ping 192.168.1.50` with replies and the ping statistics.


## Lab 2 checks

| # | Check | Method | Result |
| --- | --- | --- | --- |
| 1 | Router SNMP accepted | `show snmp` on DMT-Edge01 | Command accepted without errors. Packet Tracer shows only basic SNMP output |
| 2 | Router queried from the manager | MIB Browser, target 192.168.1.1, community `private`, version 2 | System description, uptime, interfaces, speeds, MAC addresses, administrative status and routing table retrieved |
| 3 | Switch queried from the manager | MIB Browser, target 192.168.1.2, community `private`, version 2 | Interfaces, MAC addresses, speeds and port status retrieved |
| 4 | Unsupported MIB entries | Browsing the MIB | Some entries returned “Bad Value”. Caused by Packet Tracer’s limited MIB support, not by the configuration |

## Not tested

These areas are outside what the lab documentation records.

| Area | Note |
| --- | --- |
| Voice VLAN operation | The configuration was applied and the phone was powered on. No test of voice tagging or voice and data separation is recorded |
| Isolation between VLANs | Not tested. A test would be a ping between VLANs that should fail without inter-VLAN routing |
| Inter-VLAN routing | Not configured |
| STP and link failover | STP ran at its defaults. No `show spanning-tree` output or link failure test is recorded |
| Trunk checks on Access Switch 02 and 03 | Only the core switches and Access Switch 01 are described |
| Failure behaviour of SNMP | No test with a wrong community string or an unreachable agent |
| Polling, alerts and traps | Only manual queries with the MIB Browser were done |
| Physical hardware | Everything ran in Packet Tracer |

Next: Troubleshooting and Known Issues
