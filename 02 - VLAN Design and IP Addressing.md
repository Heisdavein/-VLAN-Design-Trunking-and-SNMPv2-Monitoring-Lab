# 02 - VLAN Design and IP Addressing

## Contents

- Why VLANs
- VLAN plan
- Creating the VLANs
- Verification
- IP addressing

## Why VLANs

By default every port on a Cisco switch is in VLAN 1, so every connected device shares one broadcast domain. As devices are added, broadcast traffic grows and slows the network. VLANs divide one physical switch into several logical networks. Devices in different VLANs cannot talk directly. Their traffic has to pass through a Layer 3 device such as a router or Layer 3 switch.

| Benefit | Effect |
| --- | --- |
| Security | Devices in different VLANs are separated. User PCs cannot reach servers directly unless a Layer 3 device allows it. |
| Less broadcast traffic | Broadcasts stay inside their own VLAN. |
| Organisation | Devices can be grouped by type or department, even when connected to different switches. |
| Simpler management | Policy can be applied to a whole VLAN instead of device by device. |

## VLAN plan

Devices were grouped by **type**, not by location: all PCs together, all phones together, and so on. Grouping by type means the same rules apply to a device wherever it is plugged in, and a new printer or server only needs its port assigned to the existing VLAN.

| VLAN ID | Name | Devices | Reason |
| --- | --- | --- | --- |
| 1 | default | PCs and laptops | VLAN 1 was also the management VLAN in this lab, which kept the setup simple |
| 2 | PHONES | IP phones | Voice traffic can be prioritised and kept apart from normal data |
| 3 | PRINTERS | Printers | Less unnecessary traffic, easier control over who can print |
| 4 | SERVERS | Servers | Servers hold important services and are separated from user devices |

The plan was written down before any switch was configured. That avoided mistakes such as creating the same VLAN with different names on different switches, or placing a device in the wrong VLAN, and every switch got the same configuration.

## Creating the VLANs

A VLAN must exist in a switch’s VLAN database before a port can be assigned to it, so this was the first configuration step. The commands were entered on Core Switch 1 first:

```
enable
configure terminal

vlan 2
 name PHONES
vlan 3
 name PRINTERS
vlan 4
 name SERVERS
end

show vlan
```

The full file is `configs/lab1-vlan-trunking/all-switches-vlan-database.txt`.

| Command | Purpose |
| --- | --- |
| `enable` | Enter privileged EXEC mode |
| `configure terminal` | Enter global configuration mode |
| `vlan <id>` | Create the VLAN if it does not exist |
| `name <name>` | Give the VLAN a descriptive name |
| `show vlan` | List the VLANs stored on the switch |

### VLAN 1 cannot be renamed

The first plan was to rename VLAN 1 to PCs, matching the other VLANs. IOS refused. VLAN 1 is the reserved default VLAN and its name cannot be changed. It stays as `default`.

<img width="1376" height="768" alt="image" src="https://github.com/user-attachments/assets/6a7d9d42-d059-4d7b-b323-f3e2b06a1ea4" />
   The CLI after attempting to rename VLAN 1, showing the rejection message.


### Repeating the configuration

The same VLANs were then needed on the other four switches. The commands were saved to a text file and pasted into each switch CLI, which was faster than retyping and ensured identical names on all five switches.

### Why every switch needs every VLAN

A switch that carries a VLAN’s traffic must have that VLAN in its database. If Access Switch 01 sends VLAN 4 traffic to Core Switch 2 and Core Switch 2 has no VLAN 4, it cannot forward the traffic, and devices lose connectivity without an obvious error. A VLAN missing from one switch can break communication across the network.

VLAN 3 (printers) was created on all five switches even though only Access Switch 03 had a printer. If a printer is added elsewhere later, only the port assignment is needed.

## Verification

`show vlan` was run on all five switches. Each showed VLAN 1 (default), VLAN 2 (PHONES), VLAN 3 (PRINTERS) and VLAN 4 (SERVERS) with the correct names, matching the plan.

<img width="1376" height="768" alt="image" src="https://github.com/user-attachments/assets/dfba938b-5b1d-4f99-a89e-4a19bc97f7d7" /> 
 Output of `show vlan` on Core Switch 1: VLANs 1 to 4 with names and status active.



<img width="1376" height="768" alt="image" src="https://github.com/user-attachments/assets/9ec5b189-70d3-42ed-841f-a46aec42ad0a" />
A single image (collage or stacked captures) of `show vlan` from Core Switch 2 and Access Switch 01, 02 and 03, with each switch name visible.


## IP addressing

There was no DHCP server in the lab, so every device was given a static address. The VLAN number is used as the third octet of the address, so the VLAN of a device is visible from its IP address.

| Device | IP address | VLAN |
| --- | --- | --- |
| Server | 192.168.4.100 | 4 |
| PC 1 | 192.168.1.10 | 1 |
| PC 2 | 192.168.1.20 | 1 |
| PC 3 | 192.168.1.30 | 1 |
| PC 4 | 192.168.1.50 | 1 |
| PC 5 | 192.168.1.40 | 1 |

The subnet for each VLAN follows the same pattern: 192.168.1.0/24 (VLAN 1), 192.168.2.0/24 (VLAN 2), 192.168.3.0/24 (VLAN 3) and 192.168.4.0/24 (VLAN 4). The IP configuration shown for PC 1 in the documentation uses mask 255.255.255.0, default gateway 192.168.1.1 and DNS server 8.8.8.8.

> [!NOTE]
The documentation does not record addresses for the IP phone or printer. It also does not describe a device with the gateway address 192.168.1.1 in Lab 1. See Known Issues.

<img width="1376" height="768" alt="image" src="https://github.com/user-attachments/assets/c75ccd4e-837f-46fb-885f-e439ab6cd284" />
 PC 1 *Desktop > IP Configuration* with static addressing selected and the address, mask, gateway and DNS visible.


Next: Access Ports and Voice VLANs
