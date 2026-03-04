# Результирующая работа

## Секция Р1

**P1-Spine1**

```
service routing protocols model multi-agent
!
hostname P1-Spine1
!
interface Ethernet1
   description <to Leaf1>
   mtu 9214
   no switchport
   ip address 10.100.11.1/30
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet2
   description <to Leaf2>
   mtu 9214
   no switchport
   ip address 10.100.12.1/30
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet3
   description <to Leaf3>
   mtu 9214
   no switchport
   ip address 10.100.13.1/30
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Loopback0
   ip address 172.16.1.1/32
   ip ospf area 0.0.0.0
!
ip routing
!
router bgp 65500
   router-id 172.16.1.1
   no bgp default ipv4-unicast
   timers bgp 3 9
   distance bgp 20 200 200
   maximum-paths 2 ecmp 2
   bgp listen range 172.16.0.0/16 peer-group EVPN remote-as 65500
   neighbor EVPN peer group
   neighbor EVPN remote-as 65500
   neighbor EVPN next-hop-unchanged
   neighbor EVPN update-source Loopback0
   neighbor EVPN route-reflector-client
   neighbor EVPN send-community extended
   !
   address-family evpn
      neighbor EVPN activate
!
router ospf 1
   router-id 1.1.1.1
   auto-cost reference-bandwidth 1000
   passive-interface default
   no passive-interface Ethernet1
   no passive-interface Ethernet2
   no passive-interface Ethernet3
   max-lsa 12000
!
end
```

**P1-Spine2**

```
service routing protocols model multi-agent
!
hostname P1-Spine2
!
interface Ethernet1
   description <to Leaf1>
   mtu 9214
   no switchport
   ip address 10.100.21.1/30
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet2
   description <to Leaf2>
   mtu 9214
   no switchport
   ip address 10.100.22.1/30
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet3
   description <to Leaf3>
   mtu 9214
   no switchport
   ip address 10.100.23.1/30
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Loopback0
   ip address 172.16.1.2/32
   ip ospf area 0.0.0.0
!
ip routing
!
router bgp 65500
   router-id 172.16.1.2
   no bgp default ipv4-unicast
   timers bgp 3 9
   distance bgp 20 200 200
   maximum-paths 2 ecmp 2
   bgp listen range 172.16.0.0/16 peer-group EVPN remote-as 65500
   neighbor EVPN peer group
   neighbor EVPN remote-as 65500
   neighbor EVPN next-hop-unchanged
   neighbor EVPN update-source Loopback0
   neighbor EVPN route-reflector-client
   neighbor EVPN send-community extended
   !
   address-family evpn
      neighbor EVPN activate
!
router ospf 1
   router-id 1.1.1.2
   auto-cost reference-bandwidth 1000
   passive-interface default
   no passive-interface Ethernet1
   no passive-interface Ethernet2
   no passive-interface Ethernet3
   max-lsa 12000
!
end
```

**P1-Leaf1**

```
service routing protocols model multi-agent
!
hostname P1-Leaf1
!
vlan 10
   name VLAN10
!
vlan 20
   name VLAN20
!
vrf instance VRF1
!
vrf instance VRF2
!
interface Port-Channel8
   switchport mode trunk
   !
   evpn ethernet-segment
      identifier 0000:0000:0000:0000:1111
   lacp system-id 1111.1111.1111
!
interface Ethernet1
   description <to Spine1>
   mtu 9214
   no switchport
   ip address 10.100.11.2/30
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet2
   description <to Spine2>
   mtu 9214
   no switchport
   ip address 10.100.21.2/30
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet8
   description <to P1-Srv>
   mtu 9214
   channel-group 8 mode active
!
interface Loopback0
   ip address 172.16.2.1/32
   ip ospf area 0.0.0.0
!
interface Loopback1
   ip address 172.16.2.11/32
   ip ospf area 0.0.0.0
!
interface Vlan10
   description <VLAN10>
   vrf VRF1
   ip address virtual 192.168.10.1/24
!
interface Vlan20
   description <VLAN20>
   vrf VRF2
   ip address virtual 192.168.20.1/24
!
interface Vxlan1
   vxlan source-interface Loopback1
   vxlan udp-port 4789
   vxlan vlan 10 vni 10010
   vxlan vlan 20 vni 10020
   vxlan vrf VRF1 vni 111111
   vxlan vrf VRF2 vni 222222
   vxlan learn-restrict any
!
ip virtual-router mac-address aa:11:00:00:00:00
!
ip routing
ip routing vrf VRF1
ip routing vrf VRF2
!
router bgp 65500
   router-id 172.16.2.1
   no bgp default ipv4-unicast
   timers bgp 3 9
   distance bgp 20 200 200
   maximum-paths 2 ecmp 2
   neighbor EVPN peer group
   neighbor EVPN remote-as 65500
   neighbor EVPN update-source Loopback0
   neighbor EVPN send-community extended
   neighbor 172.16.1.1 peer group EVPN
   neighbor 172.16.1.2 peer group EVPN
   !
   vlan 10
      rd auto
      route-target both 10:10010
      redistribute learned
   !
   vlan 20
      rd auto
      route-target both 20:10020
      redistribute learned
   !
   address-family evpn
      neighbor EVPN activate
   !
   vrf VRF1
      rd 65500:1
      route-target import evpn 1:111111
      route-target export evpn 1:111111
      redistribute connected
   !
   vrf VRF2
      rd 65500:2
      route-target import evpn 2:222222
      route-target export evpn 2:222222
      redistribute connected
!
router ospf 1
   router-id 1.1.2.1
   auto-cost reference-bandwidth 1000
   passive-interface default
   no passive-interface Ethernet1
   no passive-interface Ethernet2
   max-lsa 12000
!
end
```

**P1-Leaf2**

```
service routing protocols model multi-agent
!
hostname P1-Leaf2
!
vlan 10
   name VLAN10
!
vlan 20
   name VLAN20
!
vrf instance VRF1
!
vrf instance VRF2
!
interface Port-Channel8
   switchport mode trunk
   !
   evpn ethernet-segment
      identifier 0000:0000:0000:0000:1111
   lacp system-id 1111.1111.1111
!
interface Ethernet1
   description <to Spine1>
   mtu 9214
   no switchport
   ip address 10.100.12.2/30
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet2
   description <to Spine2>
   mtu 9214
   no switchport
   ip address 10.100.22.2/30
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet8
   description <to P1-Srv>
   mtu 9214
   channel-group 8 mode active
!
interface Loopback0
   ip address 172.16.2.2/32
   ip ospf area 0.0.0.0
!
interface Loopback1
   ip address 172.16.2.22/32
   ip ospf area 0.0.0.0
!
interface Vlan10
   description <VLAN10>
   vrf VRF1
   ip address virtual 192.168.10.1/24
!
interface Vlan20
   description <VLAN20>
   vrf VRF2
   ip address virtual 192.168.20.1/24
!
interface Vxlan1
   vxlan source-interface Loopback1
   vxlan udp-port 4789
   vxlan vlan 10 vni 10010
   vxlan vlan 20 vni 10020
   vxlan vrf VRF1 vni 111111
   vxlan vrf VRF2 vni 222222
   vxlan learn-restrict any
!
ip virtual-router mac-address aa:11:00:00:00:00
!
ip routing
ip routing vrf VRF1
ip routing vrf VRF2
!
router bgp 65500
   router-id 172.16.2.2
   no bgp default ipv4-unicast
   timers bgp 3 9
   distance bgp 20 200 200
   maximum-paths 2 ecmp 2
   neighbor EVPN peer group
   neighbor EVPN remote-as 65500
   neighbor EVPN update-source Loopback0
   neighbor EVPN send-community extended
   neighbor 172.16.1.1 peer group EVPN
   neighbor 172.16.1.2 peer group EVPN
   !
   vlan 10
      rd auto
      route-target both 10:10010
      redistribute learned
   !
   vlan 20
      rd auto
      route-target both 20:10020
      redistribute learned
   !
   address-family evpn
      neighbor EVPN activate
   !
   vrf VRF1
      rd 65500:1
      route-target import evpn 1:111111
      route-target export evpn 1:111111
      redistribute connected
   !
   vrf VRF2
      rd 65500:2
      route-target import evpn 2:222222
      route-target export evpn 2:222222
      redistribute connected
!
router ospf 1
   router-id 1.1.2.2
   auto-cost reference-bandwidth 1000
   passive-interface default
   no passive-interface Ethernet1
   no passive-interface Ethernet2
   max-lsa 12000
!
end
```

**P1-Leaf3**

```
service routing protocols model multi-agent
!
hostname P1-Leaf3
!
vlan 10
   name VLAN10
!
vlan 20
   name VLAN20
!
vlan 110
   name TRANSPORT10
!
vlan 120
   name TRANSPORT20
!
vrf instance VRF1
!
vrf instance VRF2
!
interface Ethernet1
   description <to Spine1>
   mtu 9214
   no switchport
   ip address 10.100.13.2/30
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet2
   description <to Spine2>
   mtu 9214
   no switchport
   ip address 10.100.23.2/30
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet5
   description <to P2>
   mtu 9214
   no switchport
   ip address 10.10.10.1/30
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet8
   description <to BGW>
   mtu 9214
   switchport mode trunk
!
interface Loopback0
   ip address 172.16.2.3/32
   ip ospf area 0.0.0.0
!
interface Loopback1
   ip address 172.16.2.33/32
   ip ospf area 0.0.0.0
!
!
interface Vlan10
   description <VLAN10>
   vrf VRF1
   ip address virtual 192.168.10.1/24
!
interface Vlan20
   description <VLAN20>
   vrf VRF2
   ip address virtual 192.168.20.1/24
!
interface Vlan110
   description <Transport VLAN for VLAN10>
   vrf VRF1
   ip address 192.168.110.2/30
!
interface Vlan120
   description <Transport VLAN for VLAN20>
   vrf VRF2
   ip address 192.168.120.2/30
!
interface Vxlan1
   vxlan source-interface Loopback1
   vxlan udp-port 4789
   vxlan vlan 10 vni 10010
   vxlan vlan 20 vni 10020
   vxlan vlan 110 vni 10110
   vxlan vlan 120 vni 10120
   vxlan vrf VRF1 vni 111111
   vxlan vrf VRF2 vni 222222
   vxlan learn-restrict any
!
ip virtual-router mac-address aa:11:00:00:00:00
!
ip routing
ip routing vrf VRF1
ip routing vrf VRF2
!
router bgp 65500
   router-id 172.16.2.3
   no bgp default ipv4-unicast
   timers bgp 3 9
   distance bgp 20 200 200
   maximum-paths 2 ecmp 2
   neighbor EVPN peer group
   neighbor EVPN remote-as 65500
   neighbor EVPN update-source Loopback0
   neighbor EVPN send-community extended
   neighbor PtP peer group
   neighbor PtP remote-as 65501
   neighbor PtP next-hop-unchanged
   neighbor PtP update-source Loopback0
   neighbor PtP ebgp-multihop
   neighbor PtP send-community extended
   neighbor 172.16.1.1 peer group EVPN
   neighbor 172.16.1.2 peer group EVPN
   neighbor 172.17.2.3 peer group PtP
   !
   vlan 10
      rd auto
      route-target both 10:10010
      redistribute learned
   !
   vlan 20
      rd auto
      route-target both 20:10020
      redistribute learned
   !
   address-family evpn
      neighbor EVPN activate
      neighbor PtP activate
   !
   vrf VRF1
      rd 65500:1
      route-target import evpn 1:111111
      route-target export evpn 1:111111
      neighbor 192.168.110.1 remote-as 65550
      redistribute connected
      !
      address-family ipv4
         neighbor 192.168.110.1 activate
   !
   vrf VRF2
      rd 65500:2
      route-target import evpn 2:222222
      route-target export evpn 2:222222
      neighbor 192.168.120.1 remote-as 65550
      redistribute connected
      !
      address-family ipv4
         neighbor 192.168.120.1 activate
!
router ospf 1
   router-id 1.1.2.3
   auto-cost reference-bandwidth 1000
   passive-interface default
   no passive-interface Ethernet1
   no passive-interface Ethernet2
   no passive-interface Ethernet5
   max-lsa 12000
!
end
```

**P1-BGW**

```
hostname P1-BGW
!
vlan 110
   name TRANSPORT10
!
vlan 120
   name TRANSPORT20
!
interface Ethernet1
   description <to P1-Leaf3>
   mtu 9214
   switchport mode trunk
!
interface Loopback0
   ip address 100.100.100.100/32
!
interface Vlan110
   description <Transport VLAN for VLAN10>
   ip address 192.168.110.1/30
!
interface Vlan120
   description <Transport VLAN for VLAN20>
   ip address 192.168.120.1/30
!
ip routing
!
router bgp 65550
   router-id 100.100.100.100
   no bgp default ipv4-unicast
   neighbor 192.168.110.2 remote-as 65500
   neighbor 192.168.120.2 remote-as 65500
   !
   address-family ipv4
      neighbor 192.168.110.2 activate
      neighbor 192.168.120.2 activate
      network 100.100.100.100/32
!
end
```

**P1-Srv**

```
hostname P1-Srv
!
vlan 10
   name VLAN10
!
vlan 20
   name VLAN20
!
interface Port-Channel12
   switchport mode trunk
!
interface Ethernet1
   description <to Leaf1>
   mtu 9214
   channel-group 12 mode active
!
interface Ethernet2
   description <to Leaf2>
   mtu 9214
   channel-group 12 mode active
!
interface Ethernet7
   description <to VPC-10>
   switchport access vlan 10
!
interface Ethernet8
   description <to VPC-20>
   switchport access vlan 20
!
interface Vlan10
   description <VLAN10>
!
interface Vlan20
   description <VLAN20>
!
ip routing
!
end
```

## Секция Р2

**P2-Spine1**

```
service routing protocols model multi-agent
!
hostname P2-Spine1
!
interface Ethernet1
   description <to Leaf1>
   mtu 9214
   no switchport
   ip address 10.200.11.1/30
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet2
   description <to Leaf2>
   mtu 9214
   no switchport
   ip address 10.200.12.1/30
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet3
   description <to Leaf3>
   mtu 9214
   no switchport
   ip address 10.200.13.1/30
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Loopback0
   ip address 172.17.1.1/32
   ip ospf area 0.0.0.0
!
ip routing
!
router bgp 65501
   router-id 172.17.1.1
   no bgp default ipv4-unicast
   timers bgp 3 9
   distance bgp 20 200 200
   maximum-paths 2 ecmp 2
   bgp listen range 172.17.0.0/16 peer-group EVPN remote-as 65501
   neighbor EVPN peer group
   neighbor EVPN remote-as 65501
   neighbor EVPN next-hop-unchanged
   neighbor EVPN update-source Loopback0
   neighbor EVPN route-reflector-client
   neighbor EVPN send-community extended
   !
   address-family evpn
      neighbor EVPN activate
!
router ospf 1
   router-id 2.1.1.1
   auto-cost reference-bandwidth 1000
   passive-interface default
   no passive-interface Ethernet1
   no passive-interface Ethernet2
   no passive-interface Ethernet3
   max-lsa 12000
!
end
```

**P2-Spine2**

```
service routing protocols model multi-agent
!
hostname P2-Spine2
!
interface Ethernet1
   description <to Leaf1>
   mtu 9214
   no switchport
   ip address 10.200.21.1/30
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet2
   description <to Leaf2>
   mtu 9214
   no switchport
   ip address 10.200.22.1/30
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet3
   description <to Leaf3>
   mtu 9214
   no switchport
   ip address 10.200.23.1/30
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Loopback0
   ip address 172.17.1.2/32
   ip ospf area 0.0.0.0
!
ip routing
!
router bgp 65501
   router-id 172.17.1.2
   no bgp default ipv4-unicast
   timers bgp 3 9
   distance bgp 20 200 200
   maximum-paths 2 ecmp 2
   bgp listen range 172.17.0.0/16 peer-group EVPN remote-as 65501
   neighbor EVPN peer group
   neighbor EVPN remote-as 65501
   neighbor EVPN next-hop-unchanged
   neighbor EVPN update-source Loopback0
   neighbor EVPN route-reflector-client
   neighbor EVPN send-community extended
   !
   address-family evpn
      neighbor EVPN activate
!
router ospf 1
   router-id 2.1.1.2
   auto-cost reference-bandwidth 1000
   passive-interface default
   no passive-interface Ethernet1
   no passive-interface Ethernet2
   no passive-interface Ethernet3
   max-lsa 12000
!
end
```

**P2-Leaf1**

```
service routing protocols model multi-agent
!
hostname P2-Leaf1
!
no spanning-tree vlan-id 4094
!
vlan 10
   name VLAN10
!
vlan 20
   name VLAN20
!
vlan 4094
   name MLAG_PEERLINK
!
vrf instance MGMT
!
vrf instance VRF1
!
vrf instance VRF2
!
interface Port-Channel8
   description <to P2-Srv>
   switchport mode trunk
   mlag 1
!
interface Port-Channel34
   description <MLAG PEERLINK>
   switchport trunk allowed vlan 4094
   switchport mode trunk
   spanning-tree link-type point-to-point
!
interface Ethernet1
   description <to Spine1>
   mtu 9214
   no switchport
   ip address 10.200.11.2/30
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet2
   description <to Spine2>
   mtu 9214
   no switchport
   ip address 10.200.21.2/30
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet3
   description <MLAG PEERLINK>
   mtu 9214
   channel-group 34 mode active
!
interface Ethernet4
   description <MLAG PEERLINK>
   mtu 9214
   channel-group 34 mode active
!
interface Ethernet8
   description <to P1-Srv>
   mtu 9214
   channel-group 8 mode active
!
interface Loopback0
   description <eBGP VIP>
   ip address 172.17.2.1/32
   ip ospf area 0.0.0.0
!
interface Loopback1
   description <VXLAN VTEP>
   ip address 172.17.2.11/32
   ip ospf area 0.0.0.0
!
interface Management1
   description <MLAG KEEPALIVE>
   vrf MGMT
   ip address 192.168.0.1/30
!
interface Vlan10
   description <VLAN10>
   vrf VRF1
   ip address virtual 192.168.10.1/24
!
interface Vlan20
   description <VLAN20>
   vrf VRF2
   ip address virtual 192.168.20.1/24
!
interface Vlan4094
   description <MLAG PEERLINK VLAN>
   no autostate
   ip address 192.168.1.1/30
!
interface Vxlan1
   vxlan source-interface Loopback1
   vxlan udp-port 4789
   vxlan vlan 10 vni 10010
   vxlan vlan 20 vni 10020
   vxlan vrf VRF1 vni 111111
   vxlan vrf VRF2 vni 222222
   vxlan learn-restrict any
!
ip virtual-router mac-address aa:22:00:00:00:00
!
ip routing
ip routing vrf MGMT
ip routing vrf VRF1
ip routing vrf VRF2
!
mlag configuration
   domain-id MLAG-L1-L2
   local-interface Vlan4094
   peer-address 192.168.1.2
   peer-address heartbeat 192.168.0.2 vrf MGMT
   peer-link Port-Channel34
   dual-primary detection delay 1
!
router bgp 65501
   router-id 172.17.2.1
   no bgp default ipv4-unicast
   timers bgp 3 9
   distance bgp 20 200 200
   maximum-paths 2 ecmp 2
   neighbor EVPN peer group
   neighbor EVPN remote-as 65501
   neighbor EVPN update-source Loopback0
   neighbor EVPN send-community extended
   neighbor MLAG peer group
   neighbor MLAG remote-as 65501
   neighbor MLAG next-hop-self
   neighbor 172.17.1.1 peer group EVPN
   neighbor 172.17.1.2 peer group EVPN
   neighbor 192.168.1.2 peer group MLAG
   !
   vlan 10
      rd auto
      route-target both 10:10010
      redistribute learned
   !
   vlan 20
      rd auto
      route-target both 20:10020
      redistribute learned
   !
   address-family evpn
      neighbor EVPN activate
   !
   vrf VRF1
      rd 65501:1
      route-target import evpn 1:111111
      route-target export evpn 1:111111
      redistribute connected
   !
   vrf VRF2
      rd 65501:2
      route-target import evpn 2:222222
      route-target export evpn 2:222222
      redistribute connected
!
router ospf 1
   router-id 2.1.2.1
   auto-cost reference-bandwidth 1000
   passive-interface default
   no passive-interface Ethernet1
   no passive-interface Ethernet2
   max-lsa 12000
!
end
```

**P2-Leaf2**

```
service routing protocols model multi-agent
!
hostname P2-Leaf2
!
no spanning-tree vlan-id 4094
!
vlan 10
   name VLAN10
!
vlan 20
   name VLAN20
!
vlan 4094
   name MLAG_PEERLINK
!
vrf instance MGMT
!
vrf instance VRF1
!
vrf instance VRF2
!
interface Port-Channel8
   switchport mode trunk
   mlag 1
!
interface Port-Channel34
   description <MLAG PEERLINK>
   switchport trunk allowed vlan 4094
   switchport mode trunk
   spanning-tree link-type point-to-point
!
interface Ethernet1
   description <to Spine1>
   mtu 9214
   no switchport
   ip address 10.200.12.2/30
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet2
   description <to Spine2>
   mtu 9214
   no switchport
   ip address 10.200.22.2/30
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet3
   description <MLAG PEERLINK>
   mtu 9214
   channel-group 34 mode active
!
interface Ethernet4
   description <MLAG PEERLINK>
   mtu 9214
   channel-group 34 mode active
!
interface Ethernet8
   description <to P1-Srv>
   mtu 9214
   channel-group 8 mode active
!
interface Loopback0
   description <eBGP VIP>
   ip address 172.17.2.2/32
   ip ospf area 0.0.0.0
!
interface Loopback1
   description <VXLAN VTEP>
   ip address 172.17.2.22/32
   ip ospf area 0.0.0.0
!
interface Management1
   description <MLAG KEEPALIVE>
   vrf MGMT
   ip address 192.168.0.2/30
!
interface Vlan10
   description <VLAN10>
   vrf VRF1
   ip address virtual 192.168.10.1/24
!
interface Vlan20
   description <VLAN20>
   vrf VRF2
   ip address virtual 192.168.20.1/24
!
interface Vlan4094
   description <MLAG PEERLINK VLAN>
   no autostate
   ip address 192.168.1.2/30
!
interface Vxlan1
   vxlan source-interface Loopback1
   vxlan udp-port 4789
   vxlan vlan 10 vni 10010
   vxlan vlan 20 vni 10020
   vxlan vrf VRF1 vni 111111
   vxlan vrf VRF2 vni 222222
   vxlan learn-restrict any
!
ip virtual-router mac-address aa:22:00:00:00:00
!
ip routing
ip routing vrf MGMT
ip routing vrf VRF1
ip routing vrf VRF2
!
mlag configuration
   domain-id MLAG-L1-L2
   local-interface Vlan4094
   peer-address 192.168.1.1
   peer-address heartbeat 192.168.0.1 vrf MGMT
   peer-link Port-Channel34
   dual-primary detection delay 1
!
router bgp 65501
   router-id 172.17.2.2
   no bgp default ipv4-unicast
   timers bgp 3 9
   distance bgp 20 200 200
   maximum-paths 2 ecmp 2
   neighbor EVPN peer group
   neighbor EVPN remote-as 65501
   neighbor EVPN update-source Loopback0
   neighbor EVPN send-community extended
   neighbor MLAG peer group
   neighbor MLAG remote-as 65501
   neighbor MLAG next-hop-self
   neighbor 172.17.1.1 peer group EVPN
   neighbor 172.17.1.2 peer group EVPN
   !
   vlan 10
      rd auto
      route-target both 10:10010
      redistribute learned
   !
   vlan 20
      rd auto
      route-target both 20:10020
      redistribute learned
   !
   address-family evpn
      neighbor EVPN activate
   !
   vrf VRF1
      rd 65501:1
      route-target import evpn 1:111111
      route-target export evpn 1:111111
      redistribute connected
   !
   vrf VRF2
      rd 65501:2
      route-target import evpn 2:222222
      route-target export evpn 2:222222
      redistribute connected
!
router ospf 1
   router-id 2.1.2.2
   auto-cost reference-bandwidth 1000
   passive-interface default
   no passive-interface Ethernet1
   no passive-interface Ethernet2
   max-lsa 12000
!
end
```

**P2-Leaf3**

```
service routing protocols model multi-agent
!
hostname P2-Leaf3
!
vlan 10
   name VLAN10
!
vlan 20
   name VLAN20
!
vlan 210
   name TRANSPORT10
!
vlan 220
   name TRANSPORT20
!
vrf instance VRF1
!
vrf instance VRF2
!
interface Ethernet1
   description <to Spine1>
   mtu 9214
   no switchport
   ip address 10.200.13.2/30
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet2
   description <to Spine2>
   mtu 9214
   no switchport
   ip address 10.200.23.2/30
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet5
   description <to P1>
   mtu 9214
   no switchport
   ip address 10.10.10.2/30
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet8
   description <to BGW>
   mtu 9214
   switchport mode trunk
!
interface Loopback0
   ip address 172.17.2.3/32
   ip ospf area 0.0.0.0
!
interface Loopback1
   ip address 172.17.2.33/32
   ip ospf area 0.0.0.0
!
interface Vlan10
   description <VLAN10>
   vrf VRF1
   ip address virtual 192.168.10.1/24
!
interface Vlan20
   description <VLAN20>
   vrf VRF2
   ip address virtual 192.168.20.1/24
!
interface Vlan210
   description <Transport VLAN for VLAN10>
   vrf VRF1
   ip address 192.168.210.2/30
!
interface Vlan220
   description <Transport VLAN for VLAN20>
   vrf VRF2
   ip address 192.168.220.2/30
!
interface Vxlan1
   vxlan source-interface Loopback1
   vxlan udp-port 4789
   vxlan vlan 10 vni 10010
   vxlan vlan 20 vni 10020
   vxlan vlan 210 vni 10210
   vxlan vlan 220 vni 10220
   vxlan vrf VRF1 vni 111111
   vxlan vrf VRF2 vni 222222
   vxlan learn-restrict any
!
ip virtual-router mac-address aa:22:00:00:00:00
!
ip routing
ip routing vrf VRF1
ip routing vrf VRF2
!
router bgp 65501
   router-id 172.17.2.3
   no bgp default ipv4-unicast
   timers bgp 3 9
   distance bgp 20 200 200
   maximum-paths 2 ecmp 2
   neighbor EVPN peer group
   neighbor EVPN remote-as 65501
   neighbor EVPN update-source Loopback0
   neighbor EVPN send-community extended
   neighbor PtP peer group
   neighbor PtP remote-as 65500
   neighbor PtP next-hop-unchanged
   neighbor PtP update-source Loopback0
   neighbor PtP ebgp-multihop
   neighbor PtP send-community extended
   neighbor 172.16.2.3 peer group PtP
   neighbor 172.17.1.1 peer group EVPN
   neighbor 172.17.1.2 peer group EVPN
   !
   vlan 10
      rd auto
      route-target both 10:10010
      redistribute learned
   !
   vlan 20
      rd auto
      route-target both 20:10020
      redistribute learned
   !
   address-family evpn
      neighbor EVPN activate
      neighbor PtP activate
   !
   vrf VRF1
      rd 65501:1
      route-target import evpn 1:111111
      route-target export evpn 1:111111
      neighbor 192.168.210.1 remote-as 65551
      redistribute connected
      !
      address-family ipv4
         neighbor 192.168.210.1 activate
   !
   vrf VRF2
      rd 65501:2
      route-target import evpn 2:222222
      route-target export evpn 2:222222
      neighbor 192.168.220.1 remote-as 65551
      redistribute connected
      !
      address-family ipv4
         neighbor 192.168.220.1 activate
!
router ospf 1
   router-id 2.1.2.3
   auto-cost reference-bandwidth 1000
   passive-interface default
   no passive-interface Ethernet1
   no passive-interface Ethernet2
   no passive-interface Ethernet5
   max-lsa 12000
!
end
```

**P2-BGW**

```
hostname P2-BGW
!
vlan 210
   name TRANSPORT10
!
vlan 220
   name TRANSPORT20
!
interface Ethernet1
   description <to P2-Leaf3>
   mtu 9214
   switchport mode trunk
!
interface Loopback0
   ip address 200.200.200.200/32
!
interface Vlan210
   description <Transport VLAN for VLAN10>
   ip address 192.168.210.1/30
!
interface Vlan220
   description <Transport VLAN for VLAN20>
   ip address 192.168.220.1/30
!
ip routing
!
router bgp 65551
   router-id 200.200.200.200
   no bgp default ipv4-unicast
   neighbor 192.168.210.2 remote-as 65501
   neighbor 192.168.220.2 remote-as 65501
   !
   address-family ipv4
      neighbor 192.168.210.2 activate
      neighbor 192.168.220.2 activate
      network 200.200.200.200/32
!
end
```

**P2-Srv**

```
service routing protocols model ribd
!
hostname P2-Srv
!
vlan 10
   name VLAN10
!
vlan 20
   name VLAN20
!
interface Port-Channel12
   description <Port-Channel to Leaf1-Leaf2>
   switchport mode trunk
!
interface Ethernet1
   description <to Leaf1>
   mtu 9214
   channel-group 12 mode active
!
interface Ethernet2
   description <to Leaf2>
   mtu 9214
   channel-group 12 mode active
!
interface Ethernet7
   description <to VPC-11>
   switchport access vlan 10
!
interface Ethernet8
   description <to VPC-22>
   switchport access vlan 20
!
interface Vlan10
   description <VLAN10>
!
interface Vlan20
   description <VLAN20>
!
ip routing
!
end
```