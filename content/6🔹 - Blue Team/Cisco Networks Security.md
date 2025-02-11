# Cam Overflow - Protection

>[!INFO] 
>The actual way to prevent a CAM table overflow attack is to instruct each port that there's a limit to how many MAC addresses it can have, and that's done with **port security**.

**On Switches :**

```
S1#configure terminal
S1(config)#int range f0/1 - 24
S1(config-if)#no shut
S1(config-if)#switchport mode access
S1(config-if)#switchport port-security maximum 5
S1(config-if)#switchport port-security
S1(config-if)#do show port-security
exit
```

**If an attack occurs**, on the switch do :

```
show port-security
show inter status err-disabled
```

# DHCP Starvation - Protection
## Protection

**Limit rate** on every interface :

```
Switch(config)# int range f0/2 - 24
Switch(config-if-range)# ip dhcp snooping limit rate 4
Switch(config-if-range)# exit
```

**Results :**
```
Switch# show ip dhcp snooping
```

### Wireshark

![[wireshark.avif]]]

# CDP Flooding - Protection

**On Switches :**

```
S1(config-if)# no cdp run
S1# sh cdp interface int
S1# show cdp neighbors
```

- Disable on all ports except the one connected to R1.

## Why did Cisco create CDP ?

_Cisco Discovery Protocol (CDP) is a Cisco proprietary protocol designed to facilitate the network management of Cisco devices by discovering hardware and protocol information about neighboring devices. By using CDP, Network Engineers can gather information about neighboring network devices, determining the type of hardware or equipment, software version, active interfaces the device is using (whether physical or VLAN), how they are configured, and other useful information. That is quite a bit of information, and this is useful for troubleshooting and documenting the network._

# TCP Syn Flood - Protection

- [CiscoPress, Cisco Router Firewall Security: DOS Protection](https://www.ciscopress.com/articles/article.asp?p=345618&seqNum=3) 
- [Security Configuration Guide: Denial of Service Attack Prevention, Cisco IOS Release 15M&T](https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/sec_data_dos_atprvn/configuration/15-mt/sec-data-dos-atprvn-15-mt-book/sec-cfg-tcp-intercpt.html)

## TCP Intercept

**Steps on the switch**

```
enable
configure terminal
access-list access-list-number {deny | permit | remark} {host-ip-address | any | host}
ip tcp intercept list access-list-number
ip tcp intercept mode {intercept | watch}
ip tcp intercept drop-mode {oldest | random}
ip tcp intercept watch-timeout seconds
ip tcp intercept finrst-timeout seconds
ip tcp intercept connection-timeout seconds
ip tcp intercept max-incomplete low number high number
ip tcp intercept one-minute low number high number
exit
show tcp intercept connections
show tcp intercept statistics
```

# OSPF - Protection

**On routers,** setting up **authentication** process :

```
R1(config)# router ospf 1
R1(config-router)# area 0 authentication message-digest
R1(config-router)# exit
R1(config)# int g1/0
R1(config-if)# ip ospf message-digest-key 1 md5 passworD
R1(config-if)# int s0/1/1
R1(config-if)# ip ospf message-digest-key 1 md5 passworD
```

During an attack :
```
R1# show ip ospf neighbor
...
```

# ARP Spoofing - Protection

**On Switches** : DHCP snooping & rate limit.

```
S1(config)# ip dhcp snooping
S1(config)# ip dhcp snooping vlan 1
S1(config)# ip range f0/1 - 24
S1(config-if-range)# ip dhcp snooping trust
S1(config-if-range)# ip dhcp snooping limit rate 5
S1(config-if-range)# exit
```

# VLAN Hopping - Protection

```
S1(config)# int f1/1
S1(config-if)# shutdown
S1(config-if)# no shutdown
S1(config-if)# switchport mode access
S1(config-if)# switchport port-security
S1(config-if)# switchport port-security maximum 5
S1(config-if)# switchport port-security violation shutdown
S1(config-if)# exit
```

## Verification

```
S1# show mac-address-table count
```

During an attack :

![[image.avif]]
