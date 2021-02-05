<h2>Applying a Routing Policy for Importing Routes</h2>


<h3>1.1. Networking Requirements</h3>
As shown in Figure below, SW2 exchanges routing information with SW1 through OSPF and with SW3 through IS-IS. Users want SW2 to import IS-IS routes into the OSPF network. Users also want that the route to 192.168.1.0/24 on the OSPF network has a low preference and the route to 192.168.2.0/24 has a tag, which makes it easy to reference by a routing policy.

![image](https://user-images.githubusercontent.com/63696723/106989756-65326b80-67a5-11eb-83c8-b74af410b0ff.png)

<h3>1.2. Configuration Roadmap</h3>

The configuration roadmap is as follows:

- Configure a routing policy on SW2, set the cost of the route to 192.168.1.0/24 to 100, and apply the routing policy when OSPF imports IS-IS routes. The routing policy allows the route to 192.168.1.0/24 have a low preference.
- Configure a routing policy on SW2, set the tag of the route to 192.168.2.0/24 is 20, and apply the routing policy when OSPF imports IS-IS routes. In this way, the tag of the route to 192.168.2.0/24 can take effect, which makes it easy to reference by a routing policy.

<h3>1.3. Procedure</h3>

<h4>1.3.1 Basic configuration</h4>

- SW1

```
sys
sysn LSW1
vlan batch 10

int gi0/0/1
 port link-type trunk
 port trunk allow-pass vlan 10

int vlan 10
 ip address 192.168.12.1 24

```

- SW2

```
sys
sysn LSW2
vlan batch 10 20

int gi0/0/1
 port link-type trunk
 port trunk allow-pass vlan 10

int gi0/0/2
 port link-type trunk 
 port trunk allow-pass vlan 20

int vlan 10
 ip address 192.168.12.2 24

int vlan 20
 ip address 192.168.23.2 24
```

- SW3

```
sys
sysn LSW3
vlan batch 20 30 40 50

int gi0/0/1
 port link-type trunk
 port trunk allow-pass vlan 20
int gi0/0/2
 port link-type trunk
 port trunk allow-pass vlan 30

int gi0/0/3
 port link-type trunk
 port trunk allow-pass vlan 40
int gi0/0/4
 port link-type trunk
 port trunk allow-pass vlan 50

int vlan 20
 ip address 192.168.23.3 24
int vlan 30
 ip address 192.168.1.1 24
int vlan 40
 ip address 192.168.2.1 24
int vlan 50
 ip address 192.168.3.1 24
```

<h4>1.3.2. ISIS configuration</h4>

- SW2

```
isis 1
 network-entity 49.4101.0020.0200.2002.00
 is-name SW2

int vlanif20
 isis enable 1
```

- SW3

```
isis 1
 network-entity 49.4101.0030.0300.3003.00
 is-name SW3

int vlanif20
 isis enable 1
int vlanif30
 isis enable 1
int vlanif40
 isis enable 1
int vlanif50
 isis enable 1
```

<h4>1.3.3. OSPF configuration</h4>

- SW1

```
ospf 1 router-id 1.1.1.1
 area 0
  quit
 import-route isis 1 # import isis routes
 
int vlanif10
 ospf enable 1 area 0
 ospf network-type p2p
```

- SW2

```
ospf 1 router-id 2.2.2.2
 area 0
 
int vlanif10
 ospf enable 1 area 0
 ospf network-type p2p
```

Check the OSPF routing table on SW1. You can find the imported routes.

![image](https://user-images.githubusercontent.com/63696723/106989619-1ab0ef00-67a5-11eb-8684-9f18d2cca217.png)

<h4>1.3.4. Set the filtering list</h4>
- Set ACL 2002 to match 192.168.2.0/24 on SW2

```
acl 2002
 rule permit source 192.168.2.0 0.0.0.255
 quit
```

- Set an IP prefix-list name prefix-1 to match 192.168.1.0/24

```
ip ip-prefix prefix-a index 10 permit 192.168.1.0 24
```

<h4>1.3.5. Configure a routing policy</h4>

 ```
route-policy isis2ospf permit node 10
 if-match ip-prefix prefix-a
 apply cost 100
 quit
 
route-policy isis2ospf permit node 20
 if-match acl 2002
 apply tag 20
 quit
 
route-policy isis2ospf deny node 100
 quit
```

<h4>1.3.6 Apply the routing policy when routes are imported</h4>

Check the OSPF routing table on SW1, you can find that the cost of the route to 192.168.1.0/24 is 100; the tag of the route to 192.168.2.0/24 is 20; (192.168.3.0/24 was missed becasue of deny in routing policy node 100).
 
 ![image](https://user-images.githubusercontent.com/63696723/106991977-00c5db00-67aa-11eb-9b91-e0dcca0dac11.png)
 
 Done, thank ^_^!
 
