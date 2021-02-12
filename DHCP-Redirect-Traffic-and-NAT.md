<h2>Lab: DHCP, Traffic Redirection, NAT with AR2220</h2>


<h3>1.1. Networking Requirements</h3>


As shown in Figure below, company Coca Co, Ltd. has 2 departments: 

- Production (Vlan 10, represent by PC1),
- HR (Vlan 20, represent by PC2).

These 2 departments assign IP addresses using Dynamic Host Configuration Protocol (DHCP) from DHCP server
which is not connected directly to client. After users in Company able to communicate with each others in
local network, you will need configure NAT to enable users access the external network.

![image](https://user-images.githubusercontent.com/63696723/107748567-d8ab1e80-6d4b-11eb-8be4-5b3815c6bcf4.png)


<h3>1.2. Configuration Roadmap</h3>

The configuration roadmap is as follows:

- DHCP: configure DHCP relay on AR1-DHCP relay router, ip pool on AR2-DHCP server.
- NAT: AR3, confiugre traffic policy to redirect traffic from 10.0.10.0/24, 10.0.20.0/24 forward to
192.168.34.4/24 (AR4-NAT) and natting to public address (192.168.34.100 - 192.168.34.200).


<h3>1.3. Procedure</h3>


<h4>1.3.1. DHCP</h4>

- LSW1

```
vlan batch 10 20

interface Ethernet0/0/1
 port link-type access
 port default vlan 10
#
interface Ethernet0/0/2
 port link-type access
 port default vlan 20
#
interface Ethernet0/0/3
 port link-type trunk
 port trunk allow-pass vlan 10 20
#
```

- AR1-DHCP relay

```
dhcp enable
#
interface GigabitEthernet0/0/0
#
interface GigabitEthernet0/0/0.10
 dot1q termination vid 10
 ip address 10.0.10.1 255.255.255.0 
 arp broadcast enable
 dhcp select relay
 dhcp relay server-ip 172.16.0.2
#
interface GigabitEthernet0/0/0.20
 dot1q termination vid 20
 ip address 10.0.20.1 255.255.255.0 
 arp learning strict force-disable
 arp broadcast enable
 dhcp select relay
 dhcp relay server-ip 172.16.0.2
#
#
interface GigabitEthernet0/0/1
 ip address 172.16.0.1 255.255.255.0
#
ip route-static 0.0.0.0 0.0.0.0 172.16.0.2
```

- AR2-DHCP server

```
dhcp enable
#
ip pool vlan10
 gateway-list 10.0.10.1 
 network 10.0.10.0 mask 255.255.255.0 
 excluded-ip-address 10.0.10.2 10.0.10.10 
 lease day 0 hour 0 minute 3 
 dns-list 8.8.8.8 
 domain-name huawei10.com
#
ip pool vlan20
 gateway-list 10.0.20.1 
 network 10.0.20.0 mask 255.255.255.0 
 excluded-ip-address 10.0.20.200 10.0.20.254 
 dns-list 1.1.1.1 2.2.2.2 
#
#
interface GigabitEthernet0/0/0
 ip address 172.16.0.2 255.255.255.0 
 dhcp select global
#
interface GigabitEthernet0/0/1
 ip address 192.168.23.2 255.255.255.0 
 ospf enable 1 area 0.0.0.0
#
bgp 234
 peer 192.168.23.3 as-number 234 
 #
 ipv4-family unicast
  undo synchronization
  peer 192.168.23.3 enable
#
ospf 1 router-id 2.2.2.2 
 import-route static
 area 0.0.0.0 
#
ip route-static 10.0.0.0 255.255.224.0 172.16.0.1
#
```

- Verified

![image](https://user-images.githubusercontent.com/63696723/107757256-e49cdd80-6d57-11eb-8e6e-2082f0b151b7.png)


- PC1, IP configuration: DHCP

![image](https://user-images.githubusercontent.com/63696723/107753622-e3b57d00-6d52-11eb-9b1c-de3da845f207.png)

- PC2, IP configuration: DHCP

![image](https://user-images.githubusercontent.com/63696723/107753784-224b3780-6d53-11eb-85ad-fd1848cde272.png)


**DHCP configured!**


<h4>1.3.2. Traffic redirection</h4>

- AR3

```
#
acl name NAT-10 3500  
 rule 5 permit ip source 10.0.10.0 0.0.0.255 
# you can add rule 10 permit network vlan 20 here more ^_^
traffic classifier C10 operator or
 if-match acl NAT-10
#
traffic behavior B10
 redirect ip-nexthop 192.168.34.4
#
traffic policy P10
 classifier C10 behavior B10
#
#
interface GigabitEthernet0/0/0
 ip address 192.168.23.3 255.255.255.0 
 traffic-policy P10 inbound # apply traffic-policy
 ospf enable 1 area 0.0.0.0
#
interface GigabitEthernet0/0/1
 ip address 192.168.34.3 255.255.255.0 
 ospf enable 1 area 0.0.0.0
#
interface GigabitEthernet0/0/2
 ip address 192.168.35.3 255.255.255.0 
#
bgp 234
 peer 192.168.23.2 as-number 234 
 peer 192.168.35.5 as-number 500 
 #
 ipv4-family unicast
  undo synchronization
  network 192.168.23.0 
  network 192.168.34.0 
  peer 192.168.23.2 enable
  peer 192.168.23.2 next-hop-local 
  peer 192.168.35.5 enable
  peer 192.168.35.5 route-policy AS234 export
#
ospf 1 router-id 3.3.3.3 
 area 0.0.0.0 
#
route-policy AS234 permit node 10 
 if-match ip-prefix AS234 
#
ip ip-prefix AS234 index 10 permit 192.168.34.0 24
#
```

- Verified:

```
<AR3>dis traffic-policy applied-record ?
  STRING<1-31>  Name of Traffic policy
  <cr>          Please press ENTER to execute command 
<AR3>dis traffic-policy applied-record 
-------------------------------------------------
  Policy Name:   P10 
  Policy Index:  0
     Classifier:C10     Behavior:B10 
-------------------------------------------------
 *interface GigabitEthernet0/0/0
    traffic-policy P10 inbound  
      slot 0    :  success
-------------------------------------------------

```

<h4>1.3.3. NAT</h4>

- AR4-NAT

```
#
acl name NAT-10 3500  
 rule 5 permit ip source 10.0.10.0 0.0.0.255 
#
#
 nat address-group 0 192.168.34.100 192.168.34.200
#
interface GigabitEthernet0/0/0
 ip address 192.168.34.4 255.255.255.0 
 nat outbound 3500 address-group 0 
#
ip route-static 0.0.0.0 0.0.0.0 192.168.34.3
#
```

- Verified


![image](https://user-images.githubusercontent.com/63696723/107756329-a652ee80-6d56-11eb-939a-da10e3317741.png)

ping from PC1 to 5.5.5.5, and capture packet by wireshark, you will see IP source is 192.168.34.139 to dst 5.5.5.5:

![image](https://user-images.githubusercontent.com/63696723/107756462-d3070600-6d56-11eb-864c-9903e544422e.png)


Done, sorry for not detail clearly the step (tired) ^-^$$$.
