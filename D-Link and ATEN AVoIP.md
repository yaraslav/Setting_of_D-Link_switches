### 1.[Important ATEN Information for Network Administrators](#Important-ATEN-Information-for-Network-Administrators)
### 2.[Switches settings (DGS-1510/DGS-3130/DGS-3630 series)](#Switches-settings)

### Important ATEN Information for Network Administrators

In this case it uses HDMI/USB Extender `ATEN VE8952` (VE8952T/VE8952R).

See more information about setting device ane common requirements for network here: http://assets.aten.com/product/manual/VE89_Implementation_Guide.pdf

 and here: https://assets.aten.com/product/manual/ve8900-ve8950-ve8952_um_2022-05-30.pdf
 
**Addressing and Device Discovery of ATEN Devices**

ATEN devices use DHCP for addressing or auto-assign or manual-assign addressing. 

And also you can use a special ATEN software `IP Installer` for installation addresses of devices and discovery of them.

**Audio/Video (HDMI/USB) Transport and Expected Bandwidth**

1Gbit/s per port or more and in a case with cascade of switches or stacking switch - min 10Gbit/s between switches.

And also it needs to enable Jambo Frame ports and Flow Control on Ethernet.

**Synchronization**

No information

**Control and Monitoring Traffic**
ATEN Control and Managment

Address| Port| Usage| Type
---|---|---|---
225.1.0.0 | UDP 60000-6000*|Control and Monitoring via Telnet| Multicast
Device IP| TCP src.port 9000-900*|Control and Monitoring via Web-App| Unicast
Device IP| TCP dst.port any ????|Control and Monitoring via Web-App| Unicast 

**Multicast Management**

ATEN devices use multicast groups `225.10.0.0 - 225.?.?.? ` for HDMI(AV)-traffic transmition.

Aten multicasrt requirements:

- ATEN implements IGMP v2 or v3
- One IGMP Querier should be elected per VLAN
- Query intervals should be short, and time out values long.
- Use Fast Leave
- Filter Unregistered Multicast Traffic (block multicast flood)

In this case, you may need to manually add the following multicast groups to the switch's Multicast Forwarding Database to ensure the traffic makes it to all devices:

- 225.1.0.0 (01:00:5e:01:00:00)

**Energy Efficient Ethernet**

Energy Efficient Ethernet (EEE) or ‘Green ethernet’ (IEEE 802.3az) should be disabled on all ports used for Dante traffic.
EEE can result in poor synchronization performance and occasional audio dropouts.


### Switches settings 

### (DGS-1510/DGS-3130/DGS-3630 series)

First of all. 
There's some guide for DGS-1510 - via Web-interface. [See here](https://github.com/yaraslav/Setting_of_D-Link_switches/blob/main/ATEN%2BDLink-1510.pdf)

So, I'll show below the settings for DGS-1510/DGS-3130/DGS-3630 series via standart CLI (Cisco-like CLI).

First step. The common requrement is **configuring VLANs.**  
Should be configure minimum two VLANs - "managment" VLAN and "ATEN devices" VLAN. These VLANs should be different from "native" VLAN (VLAN1 ID). If required, define access and trunk interfaces.

<details> 
<summary>See configuration here</summary>

```
configure terminal
vlan 300
name managment
vlan 500
name Aten_AVoIP
exit
interface range ethernet 1/0/10-24
switchport mode access
switchport access vlan 500
exit
interface ethernet 1/0/2
switchport mode access
switchport access vlan 300
exit
interface ethernet 1/0/28
switchport mode trunk
switchport trunk allowed vlan 300,500
end
```
   
</details>

Next step:

**Optimizing for ATEN Audio-Video Traffic**

**1) EEE  - it should be disabled.** 

Energy Efficient Ethernet (EEE) - this feature is known to interrupt traffic and skew clock
synchronization. Disabling this feature is always recommended for critical live performance systems.

Disable for ports from 10 to 24
```
configure terminal
interface range ethernet 1/0/10-24
no power-saving eee
end
```
**2) Flow control**

Enable Flow control per port

```
configure terminal
interface range ethernet 1/0/10-24
flowcontrol on
end
```
**3) Jambo Frame setting**

Enable support of Jambo Frame (max.size frame - 9216 Bytes)

```
configure terminal
interface range ethernet 1/0/10-24
max-rcv-frame-size 9216
end
```
**4) QoS - setting for Clocking (PTP), AV and Control**

No info yet

**6) Multicast settings**

Enable IGMP Snooping (IGMPv2/v3):
In global configuration on a switch:
```
configure terminal
ip igmp snooping
end
```
And in a certain vlan
```
configure terminal
vlan 500
ip igmp snooping
end
```
Enable the IGMP Querier on this switch in this vlan (if using multiple switches, the core switch should be the querier).
```
configure terminal
vlan 500
ip igmp snooping querier
ip igmp snooping query-version 2
end
```
Set the Querier Interval as low as it can go, down to about 30 seconds if your switch supports it.
```
configure terminal
vlan 500
ip igmp snooping query-interval 30
end
```
Verify that IP-interface in this vlan has an IP Address in the same subnet (IP address range) as your ATEN equipment.

Set the Querier IP (vlan Ip-interface) or 0.0.0.0/Auto if the switch only has one VLAN.
```
configure terminal
interface vlan 500
ip address 192.168.5.1 255.255.255.0
end
```
If there is video equipment, set Filtered Unregistered Multicast Traffic.

If there is no video equipment, set the switch to Forward Unregistered Multicast Traffic. 

So, set multicast filtering for unregistered multicast groups...

```
configure terminal
vlan 500
multicast filtering-mode filter-unregistered
end
```
or for forwarding unregistered multicast groups.

```
configure terminal
vlan 500
multicast filtering-mode forward-unregistered
end
```

Enable Fast Leave 

```
configure terminal
vlan 500
ip igmp snooping fast-leave
end
```

**And finally,** check the confiquration

```
show running-config
```

and save this configuration!

```
copy running-config startup-config
```




