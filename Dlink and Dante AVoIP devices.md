### 1.[Important Dante Information for Network Administrators](#Important-Dante-Information-for-Network-Administrators)
### 2.[Switches settings (DGS-1510/DGS-3130/DGS-3630 series)](#Switches-settings)

###  Important Dante Information for Network Administrators

In this case it covers `ANI-series` "Dante" devices.  

**Addressing Dante Devices**

Dante devices use DHCP for addressing when available or will auto-assign an IP address in the 169.254.0.0/16 range on the primary
network and 172.31.0.0/16 on the secondary network if DHCP is not available. Dante devices continue to look for a DHCP server
even after auto-assigning an IP address. Most Dante devices support static IP addressing.

A full list of ports used by Dante is available at: www.audinate.com/learning/faqs/which-network-ports-does-dante-use

**Audio Transport and Expected Bandwidth:**

The majority of audio used in professional settings is PCM (uncompressed), sampled at 48 kHz and a bit depth (word length) of
24 bits. Dante audio is unicast by default but can be set to use multicast for cases of one-to-many distribution.
• Dante packages audio into flows to save on network overhead.
• Unicast Audio flows contain up to 4 channels. The samples-per-channel can vary between 4 and 64, depending on the
latency setting of the device. Bandwidth usage is about 6 Mbps per typical unicast audio flow.
• Bandwidth for multicast flows is dependent on the number of audio channels used. Bandwidth is about 1.5 Mbps
per channel.
• Dante audio cannot be sent over Wi-Fi.

Address| Port| Usage| Type
---|---|---|---
Device IP| UDP 14336-14591| Unicast Audio/Video| Unicast
239.255.0.0/16 UDP 4321| Multicast Audio/Video| Multicast

**Video Transport and Expected Bandwidth**

Dante video is optimized to run on Gigabit Ethernet and has a bandwidth cap of 700 Mbps. Video bandwidth is impacted by
resolution, frame rate, chroma sampling, color bit depth, compression codec used, and varies with content shown. Dante video
flows must be multicast if video is being sent to more than one destination.

**Device Discovery**

mDNS and DNS-SD are used for discovery and enumeration of other Dante devices including Dante Controller.

Address| Port| Usage| Type
---|---|---|---
224.0.0.251| 5353| mDNS| Multicast

**Synchronization**

Digital audio requires synchronization for accurate playback of audio samples. Dante uses Precision Time Protocol (PTP version 1,
IEEE 1588-2002) by default for time synchronization. This generates a few small packets, a few times per second. One clock leader
is elected on a per subnet basis that sends multicast sync and follow up messages to all followers. Follower devices send delay
requests back to the leader to determine network delay. [See more here](https://service.shure.com/articles/en_US/Knowledge/what-is-dante-clocking?r=4032&ui-knowledge-components-aura-actions.KnowledgeArticleVersionCreateDraftFromOnlineAction.createDraftFromOnlineArticle=1)

• Follower devices can be configured to send unicast delay requests to cut down on multicast traffic.

• Dante does not require PTP aware switches. In most cases Dante does not benefit from enabling boundary clock or
transparent clock on switches.
Address| Port| Usage| Type
---|---|---|---
224.0.1.129-132| UDP 319, 320| PTP| Multicast
239.254.3.3| UDP 9998| PTP Logging (if enabled)| Multicast

**Control and Monitoring Traffic**

Dante monitoring and control traffic uses the following ports: 

**External**
Address| Port| Usage| Type
---|---|---|---
224.0.0.230-233| UDP 8700-8708| Multicast Control and Monitoring| Multicast

**Internal**
Protocol| Port| Usage| Type
---|---|---|---
UDP| 4440, 4444, 4455| Audio Control| Unicast
UDP| 8751| Dante Controller metering port| Unicast
UDP| 8800| Control & Monitoring| Unicast

**QoS**

Dante as a real time media streaming service benefits from low latency and jitter on the network. QoS should be used for
prioritization of Dante clock and audio on mixed-use networks (including those with Dante Video). It is only a requirement for
Dante audio only networks if using 100 Mbps or mixed 1 Gbps/100 Mbps network infrastructure and devices.
• Dante can make use of DiffServ QoS where needed.
• Dante will tag packets, and its tags can be integrated into an existing IT network QoS scheme.
• When used, QoS must be configured with strict priority queueing.

| Priority | Usage | DSCP Label | Hex | Decimal | Binary |
| -------- | ----- | -----------| --- | ------- | ------ |
High|Time critical PTP events|CS7|0x38|56|111000
Medium|Audio, PTP v2|EF| 0x2E |46|101110|
Low| (reserved)| CS1| 0x08| 8| 001000|
None| Other traffic| Best effort| 0x00| 0| 000000|

Note:
- Dante 4.x: Clock is DSCP 46; Audio is DSCP 34 (matches AES67 standard).
- Dante 3.x: Clock is DSCP 56, Audio is DSCP 46 (matches Audinate standard).

**Multicast Management**

When Dante resides in mixed networks, those where IP video is on the same network segment, or a significant amount of
multicast audio is in use, IGMP should be used to assist with multicast management. IGMP is not a requirement for Dante audio
only networks with few or no multicast audio flows. If there is no video equipment, set the switch to Forward Unregistered Multicast Traffic on all Shure, Dante, and AES67 ports. If there is no video equipment, set the switch to Forward Unregistered Multicast Traffic on all Shure, Dante, and AES67 ports.
If video equipment is present, you will need to Filter Unregistered Multicast Traffic. 
- Dante implements IGMP v2 or v3
- One IGMP Querier should be elected per VLAN
- Query intervals should be short, and time out values long.

In this case, you may need to manually add the following multicast groups to the switch's Multicast Forwarding Database to ensure the traffic makes it to all devices:

- 224.0.0.230
- 224.0.0.231
- 224.0.0.232
- 224.0.0.233
- 224.0.0.251
- 224.0.1.129

Static filters ensure that the PTP, mDNS, Discovery, and audio traffic is always available throughout the VLAN. IGMP static filters may be required for:

- PTP traffic: 224.0.1.129 (01-00-5e-00-01-81)
- mDNS traffic: 224.0.0.251 (01-00-5e-00-00-fb)
- Shure Discovery: 239.255.254.253 (01-00-5e-7f-fe-fd)
- Dante Control: 224.0.0.230, 224.0.0.231, 224.0.0.232, and 224.0.0.233 (01-00-5e-00-00-e6, 01-00-5e-00-00-e7, 01-00-5e-00-00-e8, 01-00-5e-00-00-e9)

The specific Dante or AES67 Multicast audio addresses in use. And ensure that Filter Unregistered Multicast is not enabled. Some switches ship with this feature enabled. This blocks traffic that should be allowed (mDNS, PTP, Dante Discovery) and usually results in devices failing to appear in Dante Controller, Shure Update Utility, and Shure Designer.

If you experience intermittent audio, then run a Wireshark trace on a PC connected to the Dante network. It may show IGMP Query messages from multiple sources. Contact Shure Applications Engineering for help interpreting Wireshark traces.
Fast Leave will not harm a Dante network and is generally required for Multicast video traffic.
Avoid IGMP proxies, unless you are CERTAIN you know how it behaves.

**Energy Efficient Ethernet**

Energy Efficient Ethernet (EEE) or ‘Green ethernet’ (IEEE 802.3az) should be disabled on all ports used for Dante traffic.
EEE can result in poor synchronization performance and occasional audio dropouts.


### Switches settings 

### (DGS-1510/DGS-3130/DGS-3630 series)

First of all. 
There's some guide for DGS-1210 - via Web-interface. [See here](https://service.shure.com/articles/en_US/Knowledge/configuring-dgs-1210-switch-for-shure-devices-and-dante?r=2916&ui-knowledge-components-aura-actions.KnowledgeArticleVersionCreateDraftFromOnlineAction.createDraftFromOnlineArticle=1)

So, you can see below the settings for DGS-1510/DGS-3130/DGS-3630 series via standart CLI (Cisco-like CLI).

First step. The common requrement is **configuring VLANs.**  
Should be configure minimum two VLANs - "managment" VLAN and "Dante devices" VLAN. These VLANs should be different from "native" VLAN (VLAN1 ID). If required, define access and trunk interfaces.

<details> 
<summary>See configuration here</summary>

```
configure terminal
vlan 300
name managment
vlan 500
name Dante_AVoIP
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

**Optimizing for Dante Audio-Video Traffic**

**1) EEE  - it should be disabled.** 

Energy Efficient Ethernet (EEE) - this feature is known to interrupt traffic and skew clock
synchronization. Disabling this feature is always recommended for critical live performance systems.

Disable for ports from 21 to 24
```
configure terminal
interface range ethernet 1/0/21-24
no power-saving eee
end
```

**2) QoS - setting for Clocking (PTP), Dante Audio and Control**

a. Ensure all queues are set to Strict Priority
 ``` 
configure terminal
interface range ethernet 1/0/21-24
mls qos scheduler sp
end
 ```

b. Set all DSCP values to queue 1 (or some value below 5), for now (D-link uses queue from 0 to 7) and then for "Dante Audio-Video Traffic" set:

- DSCP value of 56 (SC7) to enter queue 7. 
- DSCP value of 46 (EF) to enter queue 6.
- DSCP value of 8 (CS1) to enter queue 5. 

```
configure terminal
interface range ethernet 1/0/21-24
mls qos map dscp-cos 0-7,9-45,47-55,57-63 to 0
mls qos map dscp-cos 56 to 7
mls qos map dscp-cos 46 to 6
mls qos map dscp-cos 8 to 5
end
```
Global setting DSCP
a. Set Trust Mode to DSCP.
b. Set Default Mode Status to Trusted (if the switch supports it)
c. Leave Ingress DSCP should be unchecked (if the switch supports it)

```
configure terminal
interface range ethernet 1/0/21-24
mls qos trust dscp 
end
```

**3) Multicast settings**

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
Verify that IP-interface in this vlan has an IP Address in the same subnet (IP address range) as your Dante/AES67 equipment.

Set the Querier IP (vlan Ip-interface) or 0.0.0.0/Auto if the switch only has one VLAN.
```
configure terminal
interface vlan 500
ip address 192.168.5.1 255.255.255.0
end
```
If there is video equipment, set Filtered Unregistered Multicast Traffic.

If there is no video equipment, set the switch to Forward Unregistered Multicast Traffic on all Shure, Dante, and AES67 ports.

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

Enable Fast Leave (Note: Fast Leave is required to support video-over-IP devices).

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





