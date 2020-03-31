# MPLS_Core_Network

![Image ipa](https://github.com/NileshChandekar/MPLS_Core_Network/blob/master/images/1.png)


* Here we are going to setup `MPLS NETWORK` for `RHOSP`
* The idea is to setup and test `DCN` topology.
* At the last you will be able to ping from `Central` to `Edge` node.
* You will be able to test `untagged` and `tagged` traffic.

### R1

  ~~~
  conf t
  hostname R1
  int lo0
  ip add 1.1.1.1 255.255.255.255
  ip ospf 1 area 0

  int GigabitEthernet1
  ip add 10.10.10.1 255.255.255.0
  no shut
  ip ospf 1 area 0

  end
  wr
  ~~~


### R2

  ~~~
  conf t
  hostname R2
  int lo0
  ip add 2.2.2.2 255.255.255.255
  ip ospf 1 are 0

  int GigabitEthernet1
  ip add 10.10.10.2 255.255.255.0
  no shut
  ip ospf 1 area 0

  int GigabitEthernet2
  ip add 20.20.20.2 255.255.255.0
  no shut
  ip ospf 1 area 0

  end
  wr
  ~~~


### R3

  ~~~
  conf t
  hostname R3
  int lo0
  ip add 3.3.3.3 255.255.255.255
  ip ospf 1 are 0

  int GigabitEthernet1
  ip add 20.20.20.1 255.255.255.0
  no shut
  ip ospf 1 area 0

  end
  wr
  ~~~

* Step 2 – Configure LDP on all the interfaces in the MPLS Core

### R1

  ~~~
  conf t
  router ospf 1
  mpls ldp autoconfig
  end
  wr
  ~~~

### R2

  ~~~
  conf t
  router ospf 1
  mpls ldp autoconfig
  end
  wr
  ~~~

### R3

  ~~~
  conf t
  router ospf 1
  mpls ldp autoconfig
  end
  wr
  ~~~


* Test on R1 / R2 and R3

  ~~~
  sh mpls interfaces
  ~~~

  ~~~
  sh mpls ldp neighbor
  ~~~

  ~~~
  R1#trace 3.3.3.3
  R3#trace 1.1.1.1
  ~~~

* Step 3 – MPLS BGP Configuration between R1 and R3

### R1

  ~~~
  conf t
  router bgp 1
  neighbor 3.3.3.3 remote-as 1
  neighbor 3.3.3.3 update-source Loopback0
  no auto-summary
   !
   address-family vpnv4
    neighbor 3.3.3.3 activate
  end
  wr
  ~~~


### R3

  ~~~
  conf t
  router bgp 1
   neighbor 1.1.1.1 remote-as 1
   neighbor 1.1.1.1 update-source Loopback0
   no auto-summary
   !
   address-family vpnv4
    neighbor 1.1.1.1 activate
  end
  wr
  ~~~


### R1

  ~~~
  sh bgp vpnv4 unicast all summary
  ~~~

* Step 4 – Add two more routers, create VRFs


### R4

  ~~~
  conf t
  hostname R4
  int lo0
  ip add 4.4.4.4 255.255.255.255
  ip ospf 2 area 2
  int GigabitEthernet1
  ip add 40.40.40.1 255.255.255.0
  ip ospf 2 area 2
  no shut
  end
  wr
  ~~~

  ~~~
  conf t
  hostname R4
  int GigabitEthernet2
  ip add 60.60.60.2 255.255.255.0
  ip ospf 2 area 2
  no shut
  end
  wr
  ~~~


### R1

  ~~~
  conf t
  int GigabitEthernet2
  no shut
  ip add 40.40.40.2 255.255.255.0
  end
  wr
  ~~~


* We are now going to start using VRF’s

### R1

  ~~~
  conf t
  ip vrf RED
  rd 4:4
  route-target both 4:4
  end
  wr
  ~~~

  ~~~
  conf t
  int GigabitEthernet2
  ip vrf forwarding RED
  end
  wr
  ~~~

  ~~~
  conf t
  int GigabitEthernet2
  ip address 40.40.40.2 255.255.255.0
  ip ospf 2 area 2
  end
  wr
  ~~~


### R4

  ~~~
  conf t
  int GigabitEthernet1
  ip ospf 2 area 2
  end
  wr
  ~~~


### R5

  ~~~
  conf t
  hostname R5
  int lo0
  ip add 5.5.5.5 255.255.255.255
  ip ospf 2 area 2
  int GigabitEthernet1
  ip add 50.50.50.1 255.255.255.0
  ip ospf 2 area 2
  no shut
  end
  wr
  ~~~

  ~~~
  conf t
  hostname R5
  int GigabitEthernet2
  ip add 70.70.70.2 255.255.255.0
  ip ospf 2 area 2
  no shut
  end
  wr
  ~~~


### R3

  ~~~
  conf t
  int GigabitEthernet2
  no shut
  ip add 50.50.50.2 255.255.255.0
  end
  wr
  ~~~


  ~~~
  conf t
  ip vrf RED
  rd 4:4
  route-target both 4:4
  end
  ~~~

  ~~~
  conf t
  int GigabitEthernet2
  ip vrf forwarding RED
  end
  ~~~

  ~~~
  conf t
  int GigabitEthernet2
  no shut
  ip add 50.50.50.2 255.255.255.0
  end
  wr
  ~~~

  ~~~
  conf t
  int GigabitEthernet2
  ip ospf 2 area 2
  end
  wr
  ~~~

* Redistribute OSPF into MP-BGP on R1

### R1

  ~~~
  conf t
  router bgp 1
  address-family ipv4 vrf RED
  redistribute ospf 2
  end
  wr
  ~~~

  ~~~
  conf t
  router ospf 2
  redistribute bgp 1 subnets
  end
  wr
  ~~~


### R3

  ~~~
  conf t
  router bgp 1
  address-family ipv4 vrf RED
  redistribute ospf 2
  end
  wr
  ~~~

  ~~~
  conf t
  router ospf 2
  redistribute bgp 1 subnets
  end
  wr
  ~~~


### CE_R1 -- (Actual R6)

  ~~~
  conf t
  hostname CE_R1
  int GigabitEthernet1
  ip address 60.60.60.1 255.255.255.0
  no shut
  ip ospf 2 area 2
  end

  conf t
  int GigabitEthernet2
  ip address 192.168.0.250 255.255.255.0
  no shut
  ip ospf 2 area 2
  #router ospf 2
  end
  wr
  ~~~

### CE_R2

  ~~~
  conf t
  hostname CE_R2
  int GigabitEthernet1
  ip address 70.70.70.1 255.255.255.0
  no shut
  ip ospf 2 area 2
  end

  conf t
  int GigabitEthernet2
  ip address 172.16.0.250 255.255.255.0
  no shut
  ip ospf 2 area 2
  #router ospf 2
  end
  wr
  ~~~

* Trunk COnfiguration

### CE_R1

  ~~~
  conf t
  interface GigabitEthernet3
  no shutdown
  end
  wr

  conf t
  interface GigabitEthernet3.710
  encapsulation dot1Q 710
  ip address 192.168.7.250 255.255.255.0
  ip ospf 2 area 2
  end
  wr
  ~~~

### CE_R2

  ~~~
  conf t
  interface GigabitEthernet3
  no shutdown
  end
  wr

  conf t
  interface GigabitEthernet3.810
  encapsulation dot1Q 810
  ip address 172.16.8.250 255.255.255.0
  ip ospf 2 area 2
  end
  wr
  ~~~

* Cumulus Switch

### Cumulus SW

  ~~~
  auto br-prov
  iface br-prov
      bridge-ports swp1 swp2
      bridge-vlan-aware no
      bridge-pvid 1
      bridge-stp off

  auto br-vlan
  iface br-vlan
      bridge-ports swp3 swp4
      bridge-pvid 1
      bridge-vids 710-750
      bridge-vlan-aware yes
      bridge-stp on
  ~~~

* Linux Nodes Interface Configurations.

### Central Node.

* IP Details

![Image ipa](https://github.com/NileshChandekar/MPLS_Core_Network/blob/master/images/2.png)

* Route Details.

![Image ipa](https://github.com/NileshChandekar/MPLS_Core_Network/blob/master/images/3.png)
