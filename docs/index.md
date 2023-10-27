# UniFi + OPNSense
I like my UniFi access point and ethernet switches. I'm less impressed with their firewall/router offerings. So I have a comodity x86 machine running OPNSense. Mixing more than one ecosystem always comes with unique challenges. So as I solve them, I'll post them here in hopes someone else may find the information useful.

[IoT VLAN](iot.md) -- The first challenge I ran into was creating a separate network for Internet of Things (IoT) devices when my firewall only has two physical ethernet ports. The short answer is VLANs. For the details, see the guide.
