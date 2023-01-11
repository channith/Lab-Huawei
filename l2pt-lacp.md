L2PT for LACP
=============


<h3>1. Network Diagram</h3>

![image](https://user-images.githubusercontent.com/63696723/211751625-0adfb4ca-a32d-4e76-83f5-665950679ff0.png)

<h3>2. Configuration</h3>

<h4>2.1. DS01</h4>

```
interface Eth-Trunk1
 description TO~DX-SHVEZE-RA02-STK01~Eth-Trunk85~Via~ME-PNHCC-RC03-CS02
 set flow-stat interval 30
 port link-type trunk
 undo port trunk allow-pass vlan 1
 port trunk allow-pass vlan 100
 stp disable
 mode lacp                                
 load-balance dst-mac                     
#

interface XGigabitEthernet1/0/7
 eth-trunk 1
 dldp enable
 storm-control broadcast min-rate 5000 max-rate 10000
 storm-control multicast min-rate 5000 max-rate 10000
 storm-control interval 1
 storm-control action block
 storm-control enable trap
#

interface XGigabitEthernet1/0/9
 eth-trunk 1
 dldp enable
 storm-control broadcast min-rate 5000 max-rate 10000
 storm-control multicast min-rate 5000 max-rate 10000
 storm-control interval 1
 storm-control action block
 storm-control enable trap
#
```

<h4>2.1. CS02</h4>

```
interface XGigabitEthernet1/0/20
 port link-type dot1q-tunnel
 port default vlan 2
 l2protocol-tunnel lacp enable
#

interface XGigabitEthernet0/0/20
 port link-type dot1q-tunnel
 port default vlan 3
 l2protocol-tunnel lacp enable
#
```


Another side:

```
interface XGigabitEthernet1/0/21
 port link-type dot1q-tunnel
 port default vlan 2
 l2protocol-tunnel lacp enable
#

interface XGigabitEthernet0/0/21
 port link-type dot1q-tunnel
 port default vlan 3
 l2protocol-tunnel lacp enable
#
```

<h4>2.1. DS02</h4>

```
interface Eth-Trunk1
 description TO~DX-SHVEZE-RA02-STK01~Eth-Trunk85~Via~ME-PNHCC-RC03-CS02
 set flow-stat interval 30
 port link-type trunk
 undo port trunk allow-pass vlan 1
 port trunk allow-pass vlan 100
 stp disable
 mode lacp                                
 load-balance dst-mac                     
#

interface XGigabitEthernet1/0/7
 eth-trunk 1
 dldp enable
 storm-control broadcast min-rate 5000 max-rate 10000
 storm-control multicast min-rate 5000 max-rate 10000
 storm-control interval 1
 storm-control action block
 storm-control enable trap
#

interface XGigabitEthernet1/0/9
 eth-trunk 1
 dldp enable
 storm-control broadcast min-rate 5000 max-rate 10000
 storm-control multicast min-rate 5000 max-rate 10000
 storm-control interval 1
 storm-control action block
 storm-control enable trap
#
```
