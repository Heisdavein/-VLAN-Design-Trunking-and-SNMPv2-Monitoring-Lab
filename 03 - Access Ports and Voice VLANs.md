# 03 - Access Ports and Voice VLANs

Creating VLANs does nothing until switch ports are assigned to them. This page covers the access port configuration on the three access switches, including the voice VLAN on Access Switch 02.

## Contents

- Access ports
- Access Switch 01
- Voice VLAN on Access Switch 02
- Access Switch 03

## Access ports

An access port belongs to exactly one VLAN. It is used for end devices such as PCs, printers, servers and IP phones, which do not understand VLAN tags. The switch puts incoming frames into the port’s VLAN and removes the VLAN information from frames it sends back to the device, so the device never sees the VLAN.

Think of a receptionist who gives each visitor a pass for the right department on arrival and collects it on exit. The visitor does not need to know how the building is organised.

## Access Switch 01

Access Switch 01 had a server on FastEthernet 0/3 and a PC on FastEthernet 0/4.

```
configure terminal

interface fastethernet 0/3
 switchport mode access
 switchport access vlan 4
 exit

interface fastethernet 0/4
 switchport mode access
 switchport access vlan 1
 exit

write memory
```

| Command | Purpose |
| --- | --- |
| `switchport mode access` | Makes the port an access port that can belong to only one VLAN |
| `switchport access vlan 4` | Places the server port in VLAN 4 |
| `switchport access vlan 1` | Places the PC port in VLAN 1 |
| `write memory` | Saves the configuration so it survives a restart |

### Why the port mode is set explicitly

Some Cisco switch ports can negotiate automatically whether to become an access or a trunk port. If two connected ports both negotiate, they can form a trunk that was not intended. Setting `switchport mode access` fixes the behaviour of the port and removes that risk, and it makes the configuration easier to read.

<img width="1376" height="768" alt="image" src="https://github.com/user-attachments/assets/9badba75-5463-4e63-9ac1-c1ed56a64342" />
 Access Switch 01 CLI showing the configuration of Fa0/3 and Fa0/4 (the commands entered, or `show running-config` for those interfaces).


## Voice VLAN on Access Switch 02

Access Switch 02 had an IP phone with a PC connected behind it. This is common in offices, where one cable runs to each desk: the phone plugs into the wall and the PC plugs into the phone. One cable then carries two kinds of traffic:

- Voice traffic from the phone
- Data traffic from the PC

Voice is sensitive to delay and jitter, and poor timing is audible on a call. Data such as downloads and web browsing tolerates small delays. If both share one VLAN, heavy data traffic can degrade calls. A voice VLAN separates them: the phone tags its voice frames with the voice VLAN ID, and the PC sends untagged frames that land in the access (data) VLAN.

```
conf t

interface fastethernet 0/1
 switchport mode access
 switchport access vlan 1
 switchport voice vlan 2
 exit

write memory
```

| Command | Effect |
| --- | --- |
| `switchport mode access` | Sets the port as an access port |
| `switchport access vlan 1` | Puts the PC’s untagged data traffic in VLAN 1 |
| `switchport voice vlan 2` | Puts the phone’s tagged voice traffic in VLAN 2 |

The port therefore carries one data VLAN and one voice VLAN at the same time. The PC needs no VLAN awareness because the phone does the tagging.

> [!NOTE]
In Packet Tracer the IP phone starts powered off. Open the device, go to the *Physical* tab and connect the phone to its power adapter before testing. The configuration alone is not enough.
> 

> **Screenshot Placeholder**
> 
> 
> Insert screenshot here:
> 
> `screenshots/access-switch-02-voice-vlan-config.png`
> 
> - **Capture:** Access Switch 02 CLI showing the Fa0/1 configuration with `switchport access vlan 1` and `switchport voice vlan 2`.
> - **Why:** Shows the data and voice VLAN split on a single port.
> - **Where:** Access Switch 02 CLI in Packet Tracer.

<img width="1376" height="768" alt="image" src="https://github.com/user-attachments/assets/12b0ab9f-aafe-4240-8dac-c6663feb4145" />
  The IP phone’s *Physical* tab with the power adapter connected and the phone powered on.


## Access Switch 03

Access Switch 03 had two PCs and one printer. Both PC ports need identical settings, so `interface range` was used to configure them together:

```
conf t

interface range fastethernet 0/1 - 2
 switchport mode access
 switchport access vlan 1
 exit

interface fastethernet 0/3
 switchport mode access
 switchport access vlan 3
 exit

write memory
```

| Command | Purpose |
| --- | --- |
| `interface range fastethernet 0/1 - 2` | Selects Fa0/1 and Fa0/2 so one set of commands applies to both |
| `switchport access vlan 1` | Places both PCs in VLAN 1 |
| `interface fastethernet 0/3` | Selects the printer port |
| `switchport access vlan 3` | Places the printer in VLAN 3 |

`interface range` saves time and keeps ports consistent, which matters more on larger switches where many ports share one configuration.

<img width="1376" height="768" alt="image" src="https://github.com/user-attachments/assets/1ce437e3-42c0-44b8-8060-7c1906760a2f" />
 Access Switch 03 CLI showing the `interface range` configuration for Fa0/1 - Fa0/2 and the printer port Fa0/3.


Next: Trunking
