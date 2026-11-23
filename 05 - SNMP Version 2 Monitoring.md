# 05 - SNMP Version 2 Monitoring

Lab 2 configured a router and a switch as SNMP agents and queried them from a monitoring server with the MIB Browser in Cisco Packet Tracer.

## Contents

- What SNMP solves
- Core concepts
- Version choice
- Lab setup
- Router configuration
- Querying the router
- Switch configuration
- Querying the switch
- SNMP in production networks

## What SNMP solves

Checking whether a port has failed on each of 80 switches by logging into every one is slow. With SNMP, one central computer collects the same information from all of them. Each network device runs an **SNMP agent** that gathers information such as CPU usage, memory usage, interface status, uptime and IP addresses. A central **SNMP manager** requests that information from the agents.

Smart electricity meters are a fair comparison: the meters (agents) report readings to the electricity company (manager), and nobody has to visit each house.

## Core concepts

| Concept | Meaning |
| --- | --- |
| SNMP agent | Software on a network device that collects information and answers requests from the manager |
| SNMP manager | The central monitoring server that requests information. In this lab, the Monitor Server |
| MIB (Management Information Base) | The database on the device that holds information such as interface status, CPU and memory |
| OID (Object Identifier) | A unique number that identifies one item in the MIB |
| Community string | The password-like value used by SNMP version 2 between manager and device. It must match on both sides |

## Version choice

| Version | Notes |
| --- | --- |
| SNMPv1 | Oldest version, very limited security |
| SNMPv2c | Faster than v1, supports retrieving larger amounts of data. Still authenticates with a community string |
| SNMPv3 | Most secure. Supports user authentication and encryption |

SNMPv2c was used because it is still common in organisations and in training material. Its community string crosses the network as plain text, so anyone who captures the traffic can read it and query the devices. SNMPv3 would be the choice for a production network that handles sensitive data. See Security Considerations.

## Lab setup

| Device | Address | Role |
| --- | --- | --- |
| Router DMT-Edge01 | 192.168.1.1 | Gateway, SNMP agent |
| Switch DMT-SW01 | 192.168.1.2 (VLAN 1) | SNMP agent |
| Monitor Server | Valid IP address | SNMP manager |

The Monitor Server has two tools: a **NetFlow Collector** (traffic and flow data, not used in this lab) and a **MIB Browser** (talks to SNMP agents, browses the MIB and retrieves device information). The MIB Browser was used throughout. Before configuring the agents, the Monitor Server was checked to confirm it already had a valid IP address, since the manager must be able to reach the devices before SNMP can work.


The Monitor Server *Desktop* tab showing the NetFlow Collector and MIB Browser icons, or its IP configuration.
> - **Why:** Shows the SNMP manager and the tools available on it.
> - **Where:** Packet Tracer, Monitor Server *Desktop* tab.

## Router configuration

```
en
conf t
snmp-server community private RO
```

`snmp-server community private RO` enables SNMP and creates a community string named `private` with **read-only** access. The monitoring server can read from the router but cannot change its configuration.

Read-only follows the principle of least privilege: a system should have only the permissions it needs. The monitoring server only collects information, so if someone gains access to it, they can read device data but cannot alter the router.

Production networks usually add location and contact details to a device. These commands were noted in the documentation but not applied, because Packet Tracer may not fully support them. The System Contact field was blank in the MIB Browser for that reason. See `optional-location-and-contact.txt`.

The configuration was checked with:

```
show snmp
```

Packet Tracer shows only basic SNMP information for this command. The result that mattered was that the command was accepted without errors.

<img width="1376" height="768" alt="image" src="https://github.com/user-attachments/assets/67ce9964-887c-4178-9a57-7257b8fc0baf" />
 DMT-Edge01 CLI with the `snmp-server community private RO` command entered.


<img width="1376" height="768" alt="image" src="https://github.com/user-attachments/assets/b46b401f-4cef-408a-b5dc-53b5eb770583" />
  Output of `show snmp` on DMT-Edge01.


## Querying the router

From the Monitor Server, the MIB Browser was set to:

| Setting | Value |
| --- | --- |
| Target IP address | 192.168.1.1 |
| Read community | private |
| SNMP version | Version 2 |

Information retrieved from the router:

| MIB category | Information |
| --- | --- |
| System Description | Router model, IOS version and software information |
| System Uptime | How long the router had been running since its last restart |
| System Contact | Blank, because it was not configured |
| Interfaces | List of network interfaces |
| Interface Speed | Speed of each interface |
| Physical Addresses | MAC address of each interface |
| Administrative Status | Whether each interface is enabled or disabled |
| Routing Table | Networks and routes known to the router |

This was done remotely over SNMP, without logging into the router CLI.

Some MIB entries returned a **“Bad Value”** message. This was not a configuration mistake. Packet Tracer supports the standard SNMP MIBs but not all the Cisco-specific MIB extensions found on real devices, which provide detail such as CPU and memory statistics.

<img width="1376" height="768" alt="image" src="https://github.com/user-attachments/assets/c9d6fb0a-6b28-4dec-83eb-80c56f186764" />
 The MIB Browser connection settings: target 192.168.1.1, read community `private`, SNMP version 2.


<img width="1376" height="768" alt="image" src="https://github.com/user-attachments/assets/a66aceab-2fe5-4060-ba15-93178c7f4443" />
MIB Browser results from the router, for example system description, uptime, interface list and routing table.


<img width="1024" height="559" alt="image" src="https://github.com/user-attachments/assets/66c0dfdb-f05a-43c7-abb0-7d2c5f1f1df1" />
  A MIB Browser query that returned “Bad Value”.


## Switch configuration

The same approach was repeated on the switch:

```
enable
conf t
snmp-server community private RO
end
write memory
```

`write memory` saves the configuration. Without it the switch would lose its SNMP settings after a reboot and the monitoring server would stop getting responses.


<img width="1024" height="572" alt="image" src="https://github.com/user-attachments/assets/cd75c21d-2397-4837-a515-7df63f3348e2" />
DMT-SW01 CLI with the SNMP configuration and `write memory`.

## Querying the switch

The MIB Browser target was changed to the switch:

| Setting | Value |
| --- | --- |
| Target IP address | 192.168.1.2 |
| Read community | private |
| SNMP version | Version 2 |

| MIB category | Information |
| --- | --- |
| Interfaces | All FastEthernet ports, Gigabit uplink ports and the VLAN 1 management interface |
| Physical Addresses | MAC address of each interface |
| Interface Speeds | Whether each port ran at FastEthernet or Gigabit speed |
| Administrative Status | Whether each port was up or down |

An administrator can check the state of every switch port from one place and see which ports are active and which have stopped working.

<img width="1024" height="572" alt="image" src="https://github.com/user-attachments/assets/c2e53e4b-3804-4e2a-840f-1a5e331807c0" />
The MIB Browser connection settings: target 192.168.1.2, read community `private`, SNMP version 2.


<img width="1024" height="559" alt="image" src="https://github.com/user-attachments/assets/5353d33d-6e70-4520-bc90-12aa2a47f908" />
MIB Browser results from the switch: interface list, MAC addresses, speeds and administrative status.


## SNMP in production networks

The Packet Tracer MIB Browser is a learning tool. Production networks use dedicated monitoring software such as PRTG Network Monitor, Nagios, OpenNMS or LibreNMS. These tools use SNMP to poll devices at regular intervals and add features the MIB Browser does not have:

| Feature | Description |
| --- | --- |
| Continuous monitoring | Devices are checked automatically at set intervals, so changes show over time |
| Automatic alerts | An alert is sent for conditions such as high CPU or a port going down |
| Historical reports | Bandwidth, CPU and similar data are stored and graphed |
| SNMP traps | Devices notify the monitoring server immediately when an important event happens |
| Central dashboard | The health of all devices is visible from one screen |

None of these were configured in the lab. The point of these features is that a failure at night can be detected and reported before users arrive in the morning.

Next: Testing and Validation
