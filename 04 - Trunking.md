# 04 - Trunking

With every device in its VLAN, the switches still could not share VLAN traffic with each other. An access port carries one VLAN, so it cannot connect switches that need to exchange several. The links between switches were configured as trunks.

## Contents

- Trunk ports
- Access switch trunks
- Native VLAN
- Allowed VLAN list
- Core switch trunks
- Verification

## Trunk ports

A trunk port carries traffic for multiple VLANs at once. Unlike an access port, it keeps the VLAN tag on each frame so the receiving switch knows which VLAN the frame belongs to. Trunks are used between switches, between a switch and a router, and between a switch and a server that uses several VLANs.

An access port is a single-lane road to one neighbourhood. A trunk is a multi-lane highway where the VLAN tag works like a lane marker.

## Access switch trunks

The uplinks from Access Switch 01 to the two core switches (Fa0/1 and Fa0/2) were configured as trunks:

```
conf t
interface range fastethernet 0/1 - 2
 switchport mode trunk
 switchport trunk native vlan 1
 switchport trunk allowed vlan 1,2,3,4
 exit
write memory
```

| Command | Purpose |
| --- | --- |
| `switchport mode trunk` | Puts the selected ports in trunk mode |
| `switchport trunk native vlan 1` | Sets VLAN 1 as the native VLAN |
| `switchport trunk allowed vlan 1,2,3,4` | Allows only VLANs 1 to 4 on the trunk |
| `write memory` | Saves the configuration |

The documentation summary states that all access-to-core uplinks were configured this way. The uplink port numbers for Access Switch 02 and 03 are not recorded (see Gaps in the source documentation).

<img width="1376" height="768" alt="image" src="https://github.com/user-attachments/assets/2874ca04-e2dd-4ce7-9978-be087e49db52" />
 Access Switch 01 CLI showing the trunk configuration of Fa0/1 - Fa0/2.

## Native VLAN

Most frames on a trunk carry a VLAN tag. One VLAN, the native VLAN, is sent **untagged**. It exists so that older devices that do not understand tagging can still communicate over the link.

Both ends of a trunk must use the same native VLAN. If one switch uses VLAN 1 and the other VLAN 99, untagged traffic is placed in the wrong VLAN, which causes communication problems and can create security risks.

VLAN 1 is already the default native VLAN. It was set explicitly on every trunk anyway so the configuration is self-documenting and does not depend on defaults.

## Allowed VLAN list

`switchport trunk allowed vlan` needs care. It does not add to the list. It **replaces** it.

| Command | Result |
| --- | --- |
| `switchport trunk allowed vlan 5` | The allowed list becomes VLAN 5 only. VLANs 1, 2, 3 and 4 are dropped from the trunk and their devices lose communication. |
| `switchport trunk allowed vlan add 5` | VLAN 5 is added and VLANs 1 to 4 stay allowed. |

The `add` keyword is what preserves the existing list. This is the command in the lab with the largest potential to cause an outage by mistake.

## Core switch trunks

The core switches are Catalyst 3560 Layer 3 switches. Unlike the 2960, the 3560 supports two trunking protocols: IEEE 802.1Q and Cisco’s older ISL. Because there is a choice, the encapsulation has to be set before trunk mode can be enabled. Without it the trunk configuration does not work.

```
conf t
interface range fastethernet 0/1 - 3
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk native vlan 1
 switchport trunk allowed vlan 1,2,3,4
 exit
write memory
```

The same configuration was applied to both Core Switch 1 and Core Switch 2.

| Command | Purpose |
| --- | --- |
| `switchport trunk encapsulation dot1q` | Selects 802.1Q, the widely supported standard for VLAN tagging |
| `switchport mode trunk` | Sets the ports to trunk |
| `switchport trunk native vlan 1` | Native VLAN 1, matching the access switches |
| `switchport trunk allowed vlan 1,2,3,4` | Allows VLANs 1 to 4 |

The 2960 supports only 802.1Q, so it does not need the encapsulation command and `switchport mode trunk` is enough. Similar switch models can need different commands.

<img width="1376" height="768" alt="image" src="https://github.com/user-attachments/assets/923b7c49-d039-4c6b-b6b3-232f235c0968" />
 Core switch CLI showing the `interface range fastethernet 0/1 - 3` trunk configuration including the `dot1q` encapsulation.

## Verification

After configuring both core switches the following commands were used:

```
show interfaces fastethernet 0/1 switchport
show running-config
```

`show interfaces fastethernet 0/1 switchport` reports whether the port is operating as a trunk, the native VLAN and the trunking protocol. `show running-config` confirms the trunk settings and allowed VLANs are present in the configuration.

The Step 3 diagram in the documentation also lists `show interfaces trunk` on Access Switch 01 for checking the uplink status.


<img width="1376" height="768" alt="image" src="https://github.com/user-attachments/assets/b59d5f31-8a80-4eac-81a6-077a65314fa2" />
  The `interface FastEthernet0/1` section of `show running-config` on a core switch with the trunk settings.

<img width="1376" height="768" alt="image" src="https://github.com/user-attachments/assets/369990fb-f910-48c8-b7a2-32edc17de73d" />
Output of `show interfaces trunk` on Access Switch 01 showing both uplinks trunking, native VLAN 1 and allowed VLANs 1 to 4.

Next: SNMP Monitoring
