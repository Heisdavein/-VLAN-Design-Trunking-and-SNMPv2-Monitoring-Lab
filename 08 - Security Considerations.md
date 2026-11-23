# 08 - Security Considerations

The labs ran in a simulator, so no real systems were exposed. This page records which security-relevant choices were made, and which weaknesses a reviewer would expect to see mentioned.

## Contents

- Measures applied in the labs
- Weaknesses and hardening not covered
- SNMP versions compared

## Measures applied in the labs

| Area | Measure | Reason |
| --- | --- | --- |
| Segmentation | Devices separated into VLANs by type | Users cannot reach servers directly unless a Layer 3 device permits it. Broadcast traffic stays inside its VLAN |
| Port mode | `switchport mode access` set explicitly on end-device ports | Prevents a port from negotiating into an unintended trunk |
| Trunk scope | `switchport trunk allowed vlan 1,2,3,4` | Only the VLANs in the design cross the trunks |
| Native VLAN | VLAN 1 set explicitly and matched on both ends of each trunk | A mismatch puts untagged traffic into the wrong VLAN and can create security risks |
| SNMP access level | Read-only community (`RO`) | Principle of least privilege. The monitoring server can read but not change device configuration |
| SNMP version choice | SNMPv2c used knowingly, with the risk recorded | It suited a controlled learning environment. SNMPv3 is the recommended choice in production |

## Weaknesses and hardening not covered

These are not mentioned in the lab documentation as problems. They are the points a network security reviewer would raise about the configuration as it stands, recorded here so the repository does not overstate the design.

| Observation | Why it matters |
| --- | --- |
| Community string is `private` | A short, widely known value that is easily guessed. SNMPv2c sends it in plain text, so it is exposed to anyone who can capture the traffic. A production string should be long and unique, or SNMPv3 should be used |
| No restriction on which hosts can query SNMP | The agents answer any host that knows the community string. Production devices normally limit queries to the monitoring server with an access list |
| VLAN 1 carries user PCs, management traffic and is the native VLAN | Best practice is a separate management VLAN and a native VLAN that no user device uses |
| Unused switch ports | The documentation does not mention shutting them down or assigning them to an unused VLAN |
| No inter-VLAN filtering | Routing between VLANs was not configured, so no policy was applied between them |

None of these were implemented, and none are claimed as done. They are listed as future improvements.

## SNMP versions compared

|  | SNMPv1 | SNMPv2c | SNMPv3 |
| --- | --- | --- | --- |
| Authentication | Community string | Community string | User-based authentication |
| Encryption | No | No | Yes |
| Community string visible on the network | Yes | Yes | Not applicable |
| Used in this lab | No | Yes | No |

Next: Lessons Learned
