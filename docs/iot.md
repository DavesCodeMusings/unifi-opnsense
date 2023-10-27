# UniFi / OPNSense IoT Network
This guide shows how to create a separate Internet of Things (IoT) WiFi VLAN for your home using a Ubiquiti UniFi access point, UniFi gigabit switch, and an OPNSense firewall installed on comodity x86 hardware.

## Why Would You Do This?
Internet of Things (IoT) devices are becoming more prevalent as Smart Home technology takes hold. Most of these devices are manufactured with their primary focus on convenience rather than security. This lax security posture can create a risk to your home network. Best practice recommendations are to segment the IoT devices off onto a separate network and use firewall rules to control what these devices can and can't access on the rest of the network.

## Why the Mix of Ubiquiti and OPNSense?
Ubiquiti UniFi products are popular with home installations by folks who want something more robust than the typical consumer grade networking gear. But unfortunately, the Unified Security Gateway (USG) firewall router never seems to be in stock and their other firewall router products carry a comparitively steep price tag. OPNSense is a solid alternative to USG that can run on commodity hardware. The only drawback is you are now dealing with two different systems from two different vendors instead of a single management interface.

## Prerequisites
This guide assumes you know how to configure UniFi network devices using the UniFi console and how to configure the OPNSense firewall. You should also have an understanding of networking concepts like: Access Points, Switches, Routers, SSIDs, IPv4 Addresses, DHCP, and VLANs.

## Hardware Used
* UniFi U6 WiFi 6 Access Point
* UniFi US8 (gen 1) POE Switch
* Beelink EQ12 dual-NIC Mini PC (firewall)
* ESP32-C3 (optional IoT client device)

## Software Used
* UniFi controller 7.5.176
* OPNSense 23.7.6 (firewall)
* MicroPython 1.21 (optional IoT client device)

## Overview of Configuration
The tutorial will use these values when setting up the devices. You can change them if you wish.

* UniFi SSID name: iotwifi
* UniFi network name: IoT
* VLAN tag: 10
* IPv4 subnet: 192.168.10.0/24
* OPNSense interface name: OPT1
* OPNSense interface IP: 192.168.10.1

OPNSense <--> UniFi Switch <--> UniFi Access Point <-WiFi-> IoT Client

## Overview of Steps
This guide covers setting up the UniFi network gear to provide the WiFi SSID and the OPNSense firewall to provide the DHCP addresses. Because there are only two network interfaces on the Beelink EQ12, the IoT network must be created as a Virtual Local Area Network (VLAN.)

1. Create a new SSID for UniFi called _iotwifi_ and test with a WiFi client.
2. Create a new Virtual Network (VLAN) for UniFi called IoT and tag it as 10.
3. Assign the _iotwifi_ SSID to the IoT network.
4. Create a new VLAN in OPNSense and tag it as 10.
5. Assign the new VLAN to the interface OPT1.
6. Configure the OPNSense DHCP service for the new VLAN.
7. Profit!

See below for detailed steps.

>Caution: Some of these steps may momentarily disrupt existing WiFi connetions, even when there is no misconfiguration.

## Creating the IoT SSID
Before creating any VLANs, first create a second SSID on your UniFi access point and test connectivity to the existing network.

1. Open the UniFi console and navigate to Settings > WiFi.
2. Use _Create New_ to add another SSID to the access point.
3. Give it a unique name and a strong password.
4. Leave the Network dropdown and any other options as default for now.
5. Add the new WiFi network.

Once this new WiFi SSID is configured, use a client device to connect to it. You can use a smartphone, a microcontroller, or an existing IoT device on your network. All you need to know at this point is the device can connect to the new SSID and get an IP address from DHCP.

## Creating the IoT VLAN in the UniFi Console
The Virtual LAN will first be created in the UniFi console and then the OPNSense firewall will be configured to match.

### Create a New UniFi Network
1. Open the UniFi console and navigate to Settings > Networks.
2. Use _New Virtual Network_ to create the VLAN.
3. Give it a network name of IoT and a VLAN ID of 10.
4. Ensure DHCP Guarding is unchecked.
5. Add the new network.

### Associate the New SSID with the New Network
1. Revisit Settings > WiFi in the UniFi console.
2. Click on the new SSID to bring up its properties.
3. Use the Network dropdown to change from the Default network to the IoT (VLAN ID=10) network.
4. Apply the changes.

After this change, the IoT test client will no longer be able to get a DHCP address, but it should still associate with the UniFi access point. You can check this by returning to the SSID list (Settings > WiFi) and looking at the number of connected clients. There should still be a connection regardless of the fact DHCP is not working yet.

## Configuring the IoT VLAN in the OPNSense Web Console
The previous steps created the VLAN in the UniFi infrastructure. Now, OPNSense must be configured to recognize the new VLAN and assign DHCP addresses to the client devices.

### Create a New OPNSense VLAN
1. Log into the OPNSense web console as an admin user.
2. Navigate to Interfaces > Other Types > VLAN.
3. Click the plus sign to create a new VLAN.
4. Ensure the Parent is the LAN interface of your firewall.
5. Use a VLAN tag of 10 (same as the UniFi VLAN tag.)
6. Leave the other settings as default.
7. Save the VLAN and Apply Changes.

### Assign the VLAN to an Interface
1. Navigate to Interfaces > Assignments
2. Verify there is a New Interface available with the value vlan01 IoT (tag 10)
3. Click the plus sign to add it.
4. Save the change.

This configuration change should result in a new OPNSense interface called OPT1.

### Configure the OPT1 Network Properties
1. In the OPNSense web console, navigate to Interfaces > OPT1.
2. Basic Configuration: Enable Interface must be checked.
3. Basic Configuration: Block private networks should be unchecked.
4. Generic Configuration: IPv4 Configuration Type needs to be set to Static IPv4.
5. Static IPv4 Configuration: IPv4 address must be 192.168.10.1/24
6. Save the configuration and Apply Changes.

Once this step is finished, return to the OPNSense Dashboard (Lobby > Dashboard) and scroll down to Interfaces and Interface Statistics. Verify the OPT1 interface exists and has an IP address of 192.168.10.1 before proceeding.
 
### Configure DHCP for the IoT network
1. In the OPNSense web console, navigate to Services > DHCPv4 > OPT1.
2. Enable DHCP server on the OPT1 interface checkbox needs to be checked.
3. Enter the range of IPs available for the DHCP server. For example: 192.168.10.50 to 192.168.10.199
4. Other DHCP parameters can be left as default.
5. Save and Apply the DHCP configuration.

After completing this step, reset the IoT client device.
Check the UniFi console under WiFi and look for a connection to the _iotwifi_ SSID.
Check the OPNSense console under Services > DHCPv4 > Leases and look for an IP address assignment.

## Troubleshooting
There are basically two things that can cause an IoT device to complain that it can't make a connection:
1. It cannot connect to the access point.
2. It cannot obtain a DHCP address.

To verify connection to the access point, use the UniFi console. Navigate to Settings to view the UniFi access point SSID list. Pay attention to the column labeled Clients (Peak). Keep in mind you may need to refresh the browser to get a current and accurate count of clients. If you see a connection, it means your IoT device has the correct SSID and password.

You may also want to use the UniFi console's Topology view to see if your IoT device appears. My MicroPython device appears as `mpy-esp32c3` in the topology view. With the client details switched on, you will also see the SSID the client device is associated with.

To verify a DHCP address lease for the device, use the OPNSense console. Navigate to Services > DHCPv4 > Leases and look for an IP address assignment in the range of your IoT subnet. (192.168.10.50 to 192.168.10.199 if you're following the examples here.)

To help narrow down the problem, use the UniFi console. Navigate to Settings and assign the SSID to the Default Network. If your IoT device can connect and get an IP address, you know the SSID and password is okay. You can then focus on troubleshooting DHCP and VLAN.

First, use the UniFi console to move your IoT SSID back to the IoT network before you continue.

Next, navigate to UniFi Devices in the UniFi console. Select the switch between the UniFi access point and the OPNSense firewall. Use Port Manager to examine the switch ports where these devices attach. Ensure the ports have Traffic Restriction turned off. (Users with advanced configurations may need to add the IoT VLAN to the allow list, but for most, disabling traffic restriction is the best option.)

In the OPNSense console, access the Lobby > Dashboard. Examine Interfaces to ensure the IP configuration is correct and the OPT1 interface is up (shown in green.) Also check the Interface Statistics. The OPT1 interface should show packets in and packets. It will be a small number, but should be greater than zero. If no packets are flowing, double check the interface setup.

You can also check the OPNSense configuration from the SSH interface using the command `ifconfig`. You should see your IoT VLAN as part of the output.

## Further Security Hardening
The whole reason for creating a separate IoT network is to improve your network's security. There are several more steps you can take to this end. This includes creating firewall rules in OPNSense and restricting VLANs on your UniFi switch ports. What you configure depends on a number of factors and is beyond the scope of this document. This is just a friendly reminder that you've taken the first step by segmenting your network to isolate IoT devices, but you're not done yet.
