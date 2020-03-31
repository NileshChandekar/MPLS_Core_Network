# MPLS_Core_Network

![Image ipa](/home/nchandek/redhat/githubprojects/MPLS_Core_Network/images/1.png)


* Here we are going to setup `MPLS NETWORK` for `RHOSP`
* The idea is to setup and test `DCN` topology.
* At the last you will be able to ping from `Central` to `Edge` node.
* You will be able to test `untagged` and `tagged` traffic.

# R1

~~~
R1#sh run
Building configuration...

Current configuration : 4112 bytes
!
! Last configuration change at 16:24:16 UTC Wed Mar 18 2020

hostname R1

ip vrf RED
 rd 4:4
 route-target export 4:4
 route-target import 4:4

interface Loopback0
 ip address 1.1.1.1 255.255.255.255
 ip ospf 1 area 0
!
interface GigabitEthernet1
 ip address 10.10.10.1 255.255.255.0
 ip ospf 1 area 0
 negotiation auto
 no mop enabled
 no mop sysid
!
interface GigabitEthernet2
 ip vrf forwarding RED
 ip address 40.40.40.2 255.255.255.0
 ip ospf 2 area 2
 negotiation auto
 no mop enabled
 no mop sysid
!
router ospf 2 vrf RED
 redistribute bgp 1 subnets
!
router ospf 1
 mpls ldp autoconfig
!
router bgp 1
 bgp log-neighbor-changes
 neighbor 3.3.3.3 remote-as 1
 neighbor 3.3.3.3 update-source Loopback0
 !
 address-family vpnv4
  neighbor 3.3.3.3 activate
  neighbor 3.3.3.3 send-community extended
 exit-address-family
 !
 address-family ipv4 vrf RED
  redistribute ospf 2
 exit-address-family
!
~~~

# R2
