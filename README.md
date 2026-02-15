# ngn-project-
# Configuration Backbone IP/MPLS VPN

## Architecture

```
                    +-------+
                    |  P1   |
                   /  3.3.3.3 \
                  /            \
+------+    +------+          +------+    +------+
| CE11 |----| PE1  |          | PE2  |----| CE12 |
+------+    | 1.1.1.1         | 2.2.2.2   +------+
            +------+          +------+
+------+       |    \        /    |       +------+
| CE21 |-------+     \      /     +-------| CE22 |
+------+              \    /              +------+
                       +--+
                       | P2 |
                       |4.4.4.4|
                       +------+
```

## Plan d'adressage

### Backbone (10.1.1.0/24)

| Lien | Sous-reseau |
|------|-------------|
| PE1-P1 | 10.1.1.0/30 |
| PE1-P2 | 10.1.1.4/30 |
| PE2-P2 | 10.1.1.8/30 |
| PE2-P1 | 10.1.1.12/30 |
| P1-P2 | 10.1.1.20/30 |

### Loopbacks

| Routeur | Loopback0 |
|---------|-----------|
| PE1 | 1.1.1.1/32 |
| PE2 | 2.2.2.2/32 |
| P1 | 3.3.3.3/32 |
| P2 | 4.4.4.4/32 |
| CE11 | 172.16.11.11/32 |
| CE12 | 172.16.12.12/32 |
| CE21 | 172.16.21.21/32 |
| CE22 | 172.16.22.22/32 |

### WAN Client (192.168.1.0/24)

| Lien | Sous-reseau |
|------|-------------|
| CE11-PE1 | 192.168.1.0/30 |
| CE21-PE1 | 192.168.1.4/30 |
| CE12-PE2 | 192.168.1.8/30 |
| CE22-PE2 | 192.168.1.12/30 |

---

# CONFIGURATION DES ROUTEURS

## 1. Configuration de P1

```cisco
P1#conf t

! Interfaces
P1(config)#interface Loopback0
P1(config-if)#ip address 3.3.3.3 255.255.255.255

P1(config)#interface GigabitEthernet0/0
P1(config-if)#ip address 10.1.1.21 255.255.255.252
P1(config-if)#no shutdown

P1(config)#interface GigabitEthernet0/1
P1(config-if)#ip address 10.1.1.2 255.255.255.252
P1(config-if)#no shutdown

P1(config)#interface GigabitEthernet0/2
P1(config-if)#ip address 10.1.1.14 255.255.255.252
P1(config-if)#no shutdown

! OSPF
P1(config)#router ospf 1
P1(config-router)#router-id 3.3.3.3
P1(config-router)#network 3.3.3.3 0.0.0.0 area 0
P1(config-router)#network 10.1.1.20 0.0.0.3 area 0
P1(config-router)#network 10.1.1.0 0.0.0.3 area 0
P1(config-router)#network 10.1.1.12 0.0.0.3 area 0

! MPLS
P1(config)#ip cef
P1(config)#mpls ip
P1(config)#mpls label protocol ldp
P1(config)#mpls ldp router-id Loopback0 force

P1(config)#interface GigabitEthernet0/0
P1(config-if)#mpls ip
P1(config)#interface GigabitEthernet0/1
P1(config-if)#mpls ip
P1(config)#interface GigabitEthernet0/2
P1(config-if)#mpls ip
P1(config-if)#end
P1#write
```

---

## 2. Configuration de P2

```cisco
P2#conf t

! Interfaces
P2(config)#interface Loopback0
P2(config-if)#ip address 4.4.4.4 255.255.255.255

P2(config)#interface GigabitEthernet0/0
P2(config-if)#ip address 10.1.1.22 255.255.255.252
P2(config-if)#no shutdown

P2(config)#interface GigabitEthernet0/1
P2(config-if)#ip address 10.1.1.10 255.255.255.252
P2(config-if)#no shutdown

P2(config)#interface GigabitEthernet0/2
P2(config-if)#ip address 10.1.1.6 255.255.255.252
P2(config-if)#no shutdown

! OSPF
P2(config)#router ospf 1
P2(config-router)#router-id 4.4.4.4
P2(config-router)#network 4.4.4.4 0.0.0.0 area 0
P2(config-router)#network 10.1.1.20 0.0.0.3 area 0
P2(config-router)#network 10.1.1.8 0.0.0.3 area 0
P2(config-router)#network 10.1.1.4 0.0.0.3 area 0

! MPLS
P2(config)#ip cef
P2(config)#mpls ip
P2(config)#mpls label protocol ldp
P2(config)#mpls ldp router-id Loopback0 force

P2(config)#interface GigabitEthernet0/0
P2(config-if)#mpls ip
P2(config)#interface GigabitEthernet0/1
P2(config-if)#mpls ip
P2(config)#interface GigabitEthernet0/2
P2(config-if)#mpls ip
P2(config-if)#end
P2#write
```

---

## 3. Configuration de PE1

```cisco
PE1#conf t

! Interfaces Backbone
PE1(config)#interface Loopback0
PE1(config-if)#ip address 1.1.1.1 255.255.255.255

PE1(config)#interface GigabitEthernet0/0
PE1(config-if)#ip address 10.1.1.1 255.255.255.252
PE1(config-if)#no shutdown

PE1(config)#interface GigabitEthernet0/1
PE1(config-if)#ip address 10.1.1.5 255.255.255.252
PE1(config-if)#no shutdown

! OSPF Backbone
PE1(config)#router ospf 1
PE1(config-router)#router-id 1.1.1.1
PE1(config-router)#network 1.1.1.1 0.0.0.0 area 0
PE1(config-router)#network 10.1.1.0 0.0.0.3 area 0
PE1(config-router)#network 10.1.1.4 0.0.0.3 area 0

! MPLS
PE1(config)#ip cef
PE1(config)#mpls ip
PE1(config)#mpls label protocol ldp
PE1(config)#mpls ldp router-id Loopback0 force

PE1(config)#interface GigabitEthernet0/0
PE1(config-if)#mpls ip
PE1(config)#interface GigabitEthernet0/1
PE1(config-if)#mpls ip

! Creation VRF
PE1(config)#ip vrf VPN_Customer1
PE1(config-vrf)#rd 100:1
PE1(config-vrf)#route-target both 100:1

PE1(config)#ip vrf VPN_Customer2
PE1(config-vrf)#rd 100:2
PE1(config-vrf)#route-target both 100:2

! Interfaces Client avec VRF
PE1(config)#interface GigabitEthernet0/2
PE1(config-if)#ip vrf forwarding VPN_Customer1
PE1(config-if)#ip address 192.168.1.1 255.255.255.252
PE1(config-if)#no shutdown

PE1(config)#interface GigabitEthernet0/3
PE1(config-if)#ip vrf forwarding VPN_Customer2
PE1(config-if)#ip address 192.168.1.5 255.255.255.252
PE1(config-if)#no shutdown

! MP-BGP
PE1(config)#router bgp 100
PE1(config-router)#no bgp default ipv4-unicast
PE1(config-router)#neighbor 2.2.2.2 remote-as 100
PE1(config-router)#neighbor 2.2.2.2 update-source Loopback0
PE1(config-router)#address-family vpnv4 unicast
PE1(config-router-af)#neighbor 2.2.2.2 activate
PE1(config-router-af)#neighbor 2.2.2.2 send-community both
PE1(config-router-af)#exit
PE1(config-router)#address-family ipv4 vrf VPN_Customer1
PE1(config-router-af)#redistribute ospf 100
PE1(config-router-af)#exit
PE1(config-router)#address-family ipv4 vrf VPN_Customer2
PE1(config-router-af)#redistribute ospf 200
PE1(config-router-af)#exit

! OSPF pour VRF
PE1(config)#router ospf 100 vrf VPN_Customer1
PE1(config-router)#router-id 1.1.1.100
PE1(config-router)#redistribute bgp 100 subnets
PE1(config-router)#network 192.168.1.0 0.0.0.3 area 11

PE1(config)#router ospf 200 vrf VPN_Customer2
PE1(config-router)#router-id 1.1.1.200
PE1(config-router)#redistribute bgp 100 subnets
PE1(config-router)#network 192.168.1.4 0.0.0.3 area 21
PE1(config-router)#end
PE1#write
```

---

## 4. Configuration de PE2

```cisco
PE2#conf t

! Interfaces Backbone
PE2(config)#interface Loopback0
PE2(config-if)#ip address 2.2.2.2 255.255.255.255

PE2(config)#interface GigabitEthernet0/0
PE2(config-if)#ip address 10.1.1.9 255.255.255.252
PE2(config-if)#no shutdown

PE2(config)#interface GigabitEthernet0/1
PE2(config-if)#ip address 10.1.1.13 255.255.255.252
PE2(config-if)#no shutdown

! OSPF Backbone
PE2(config)#router ospf 1
PE2(config-router)#router-id 2.2.2.2
PE2(config-router)#network 2.2.2.2 0.0.0.0 area 0
PE2(config-router)#network 10.1.1.8 0.0.0.3 area 0
PE2(config-router)#network 10.1.1.12 0.0.0.3 area 0

! MPLS
PE2(config)#ip cef
PE2(config)#mpls ip
PE2(config)#mpls label protocol ldp
PE2(config)#mpls ldp router-id Loopback0 force

PE2(config)#interface GigabitEthernet0/0
PE2(config-if)#mpls ip
PE2(config)#interface GigabitEthernet0/1
PE2(config-if)#mpls ip

! Creation VRF
PE2(config)#ip vrf VPN_Customer1
PE2(config-vrf)#rd 100:1
PE2(config-vrf)#route-target both 100:1

PE2(config)#ip vrf VPN_Customer2
PE2(config-vrf)#rd 100:2
PE2(config-vrf)#route-target both 100:2

! Interfaces Client avec VRF
PE2(config)#interface GigabitEthernet0/2
PE2(config-if)#ip vrf forwarding VPN_Customer1
PE2(config-if)#ip address 192.168.1.9 255.255.255.252
PE2(config-if)#no shutdown

PE2(config)#interface GigabitEthernet0/3
PE2(config-if)#ip vrf forwarding VPN_Customer2
PE2(config-if)#ip address 192.168.1.13 255.255.255.252
PE2(config-if)#no shutdown

! MP-BGP
PE2(config)#router bgp 100
PE2(config-router)#no bgp default ipv4-unicast
PE2(config-router)#neighbor 1.1.1.1 remote-as 100
PE2(config-router)#neighbor 1.1.1.1 update-source Loopback0
PE2(config-router)#address-family vpnv4 unicast
PE2(config-router-af)#neighbor 1.1.1.1 activate
PE2(config-router-af)#neighbor 1.1.1.1 send-community both
PE2(config-router-af)#exit
PE2(config-router)#address-family ipv4 vrf VPN_Customer1
PE2(config-router-af)#redistribute ospf 100
PE2(config-router-af)#exit
PE2(config-router)#address-family ipv4 vrf VPN_Customer2
PE2(config-router-af)#redistribute ospf 200
PE2(config-router-af)#exit

! OSPF pour VRF
PE2(config)#router ospf 100 vrf VPN_Customer1
PE2(config-router)#router-id 2.2.2.100
PE2(config-router)#redistribute bgp 100 subnets
PE2(config-router)#network 192.168.1.8 0.0.0.3 area 12

PE2(config)#router ospf 200 vrf VPN_Customer2
PE2(config-router)#router-id 2.2.2.200
PE2(config-router)#redistribute bgp 100 subnets
PE2(config-router)#network 192.168.1.12 0.0.0.3 area 22
PE2(config-router)#end
PE2#write
```

---

## 5. Configuration de CE11

```cisco
CE11#conf t

CE11(config)#interface Loopback0
CE11(config-if)#ip address 172.16.11.11 255.255.255.255

CE11(config)#interface GigabitEthernet0/1
CE11(config-if)#ip address 192.168.1.2 255.255.255.252
CE11(config-if)#no shutdown

CE11(config)#router ospf 1
CE11(config-router)#router-id 172.16.11.11
CE11(config-router)#network 192.168.1.0 0.0.0.3 area 11
CE11(config-router)#network 172.16.11.11 0.0.0.0 area 11
CE11(config-router)#end
CE11#write
```

---

## 6. Configuration de CE12

```cisco
CE12#conf t

CE12(config)#interface Loopback0
CE12(config-if)#ip address 172.16.12.12 255.255.255.255

CE12(config)#interface GigabitEthernet0/0
CE12(config-if)#ip address 192.168.1.10 255.255.255.252
CE12(config-if)#no shutdown

CE12(config)#router ospf 1
CE12(config-router)#router-id 172.16.12.12
CE12(config-router)#network 192.168.1.8 0.0.0.3 area 12
CE12(config-router)#network 172.16.12.12 0.0.0.0 area 12
CE12(config-router)#end
CE12#write
```

---

## 7. Configuration de CE21

```cisco
CE21#conf t

CE21(config)#interface Loopback0
CE21(config-if)#ip address 172.16.21.21 255.255.255.255

CE21(config)#interface GigabitEthernet0/0
CE21(config-if)#ip address 192.168.1.6 255.255.255.252
CE21(config-if)#no shutdown

CE21(config)#router ospf 1
CE21(config-router)#router-id 172.16.21.21
CE21(config-router)#network 192.168.1.4 0.0.0.3 area 21
CE21(config-router)#network 172.16.21.21 0.0.0.0 area 21
CE21(config-router)#end
CE21#write
```

---

## 8. Configuration de CE22

```cisco
CE22#conf t

CE22(config)#interface Loopback0
CE22(config-if)#ip address 172.16.22.22 255.255.255.255

CE22(config)#interface GigabitEthernet0/0
CE22(config-if)#ip address 192.168.1.14 255.255.255.252
CE22(config-if)#no shutdown

CE22(config)#router ospf 1
CE22(config-router)#router-id 172.16.22.22
CE22(config-router)#network 192.168.1.12 0.0.0.3 area 22
CE22(config-router)#network 172.16.22.22 0.0.0.0 area 22
CE22(config-router)#end
CE22#write
```

---

# COMMANDES DE VERIFICATION ET TEST

## Tests de connectivite de base

```cisco
show ip interface brief
ping <adresse_voisin>
```

## Tests OSPF

```cisco
show ip ospf neighbor
show ip ospf interface brief
show ip route ospf
show ip ospf database
```

## Tests MPLS / LDP

```cisco
show mpls interfaces
show mpls ldp neighbor
show mpls ldp bindings
show mpls forwarding-table
show mpls ip binding
traceroute <destination> source <source>
```

## Tests VRF

```cisco
show ip vrf
show ip vrf detail
show ip vrf interfaces
show ip route vrf VPN_Customer1
show ip route vrf VPN_Customer2
```

## Tests BGP VPNv4

```cisco
show ip bgp vpnv4 all summary
show ip bgp vpnv4 all
show ip bgp vpnv4 vrf VPN_Customer1
show ip bgp vpnv4 vrf VPN_Customer2
```

## Tests OSPF VRF

```cisco
show ip ospf 100 neighbor
show ip ospf 200 neighbor
```

## Tests finaux VPN

### VPN Customer 1 (CE11 - CE12)

```cisco
CE11#ping 172.16.12.12 source 172.16.11.11
CE12#ping 172.16.11.11 source 172.16.12.12
```

### VPN Customer 2 (CE21 - CE22)

```cisco
CE21#ping 172.16.22.22 source 172.16.21.21
CE22#ping 172.16.21.21 source 172.16.22.22
```

## Test d'isolation VPN (doit echouer)

```cisco
CE11#ping 172.16.21.21
CE11#ping 172.16.22.22
```

---

# CABLAGE GNS3

| Source | Interface | Destination | Interface |
|--------|-----------|-------------|-----------|
| CE11 | Gi0/1 | PE1 | Gi0/2 |
| CE12 | Gi0/0 | PE2 | Gi0/2 |
| CE21 | Gi0/0 | PE1 | Gi0/3 |
| CE22 | Gi0/0 | PE2 | Gi0/3 |
| PE1 | Gi0/0 | P1 | Gi0/1 |
| PE1 | Gi0/1 | P2 | Gi0/2 |
| PE2 | Gi0/0 | P2 | Gi0/1 |
| PE2 | Gi0/1 | P1 | Gi0/2 |
| P1 | Gi0/0 | P2 | Gi0/0 |

---

# CRITERES DE SUCCES

| Test | Resultat attendu |
|------|------------------|
| ping 172.16.12.12 depuis CE11 | 100% success |
| ping 172.16.22.22 depuis CE21 | 100% success |
| ping 172.16.21.21 depuis CE11 | Echec (isolation VPN) |
| show ip bgp vpnv4 all summary | Etat Established |
| show mpls ldp neighbor | Voisins UP |

---

*Document genere pour la configuration du Backbone IP/MPLS VPN*

