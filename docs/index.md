# UniFi + OPNSense
I like my UniFi access point and ethernet switches. I'm less impressed with their firewall/router offerings. So I have a comodity x86 machine running OPNSense. Mixing more than one ecosystem always comes with unique challenges. So as I solve them, I'll post solutions here in hopes someone else may find the information useful.

## Network Topology
![Network Topology Diagram](NetworkTopology.png)

_Hastily constructed network diagram using LibreOffice and Material Design icons_

My home network consists of an 8-port UniFi POE switch at its core.
* One POE port powers the U6 Lite access point
* The remaining three POE ports serve as uplinks for Flex Mini 5-port switches attached to home entertainment, home automation, and various Linux servers.
* Internet service comes through the OPNSense firewall.

## Setup Details
[IoT VLAN](iot.md) -- The first challenge I ran into was creating a separate network for Internet of Things (IoT) devices when my firewall only has two physical ethernet ports. The short answer is VLANs. For the details, see the guide.
