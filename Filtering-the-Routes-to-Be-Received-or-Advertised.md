<h2>Filtering the Routes to Be Received or Advertised</h2>

As shown in Figure below, on the network where OSPF runs, SwitchA receives routes from the Internet, and provides these routes for the OSPF network. Users want devices on the OSPF network to access only the network segments 172.16.17.0/24, 172.16.18.0/24, and 172.16.19.0/24, and SwitchC to access only the network segment 172.16.18.0/24.

![image](https://user-images.githubusercontent.com/63696723/106857890-67d48880-66f3-11eb-8a9c-64302d760ae0.png)

<h3>hConfiguration Roadmap</h3>

The configuration roadmap is as follows:

- Configure a routing policy on SwitchA and apply the routing policy during route advertisement. When routes are advertised, the routing policy allows SwitchA to provide routes from network segments 172.16.17.0/24, 172.16.18.0/24, and 172.16.19.0/24 for SwitchB, and allows devices on the OSPF network to access these three network segments.

- Configure a routing policy on SwitchC and apply the routing policy during route importing. When routes are imported, the routing policy allows SwitchC to receive only the routes from the network segment 172.16.18.0/24 and access this network segment.

<h3>Procedure</h3>

<h4>Basic configuration</h4>

- Switch A
```
# Create vlan
vlan batch 10

# Create SVI
interface Vlanif10
 ip address 192.168.1.1 255.255.255.0
 quit
 
interface LoopBack0
 ip address 1.1.1.1 255.255.255.255
 quit 
 
 # Allow vlan on physical interface
 interface GigabitEthernet0/0/1
 port link-type access
 port default vlan 10
 
 ```
 
 - Switch B
 ```
 vlan batch 10 20
 
 interface Vlanif10
 ip address 192.168.1.2 255.255.255.0
 quit
 
 interface Vlanif20
 ip address 192.168.2.2 255.255.255.0
 quit
 
 interface GigabitEthernet0/0/1
 port link-type access
 port default vlan 10
#
interface GigabitEthernet0/0/2
 port link-type access
 port default vlan 20
 quit
```

- Switch C
```
vlan batch 20

interface Vlanif20
 ip address 192.168.2.3 255.255.255.0
 quit
 
interface GigabitEthernet0/0/1
 port link-type access
 port default vlan 20
 quit
 
interface LoopBack0
 ip address 3.3.3.3 255.255.255.255
 quit
```

<h4>OSPF configuration (single area)</h4>

- Switch A
```
ospf 1 router-id 1.1.1.1
 area 0.0.0.1
 quit
interface LoopBack0
 ospf enable 1 area 0.0.0.1
 quit
interface Vlanif10
 ospf enable 1 area 0.0.0.1
 quit
```

- Switch B,C enable ospf are the same as the configuration of SwitchA

