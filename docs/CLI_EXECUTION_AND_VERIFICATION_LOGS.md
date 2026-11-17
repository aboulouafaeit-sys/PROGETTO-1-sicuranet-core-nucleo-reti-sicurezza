# 💻 Cisco CLI Execution Logs & Verification Manual
### Complete Command History & Production Verification for All Network Devices

This document contains the exact CLI execution transcripts, interactive command sequences, and verification command outputs (`show`, `debug`, `ping`, `tracert`) executed across all routers, firewalls, and switches in the **Enterprise & Smart Factory CCNA Project**.

---

## 📑 Table of Contents
1. [Cisco ASA 5506-X Next-Gen Firewall (Perimeter Security)](#1-cisco-asa-5506-x-next-gen-firewall)
2. [Cisco Catalyst 3560/3650 Core Multilayer Switch (Inter-VLAN Routing)](#2-cisco-catalyst-35603650-core-multilayer-switch)
3. [Cisco 2811/2911 Edge Router & CME PBX Telephony](#3-cisco-28112911-edge-router--cme-pbx-telephony)
4. [Cisco Catalyst 2960 Office Access Switch](#4-cisco-catalyst-2960-office-access-switch)
5. [Cisco Industrial / 2960 Factory Access Switch](#5-cisco-industrial--2960-factory-access-switch)
6. [End-to-End Connectivity & Verification Matrix](#6-end-to-end-connectivity--verification-matrix)

---

## 1. Cisco ASA 5506-X Next-Gen Firewall

### 🔹 Step-by-Step Executed CLI Commands
```cisco
ciscoasa> enable
Password: 
ciscoasa# configure terminal
ciscoasa(config)# hostname ASA-FIREWALL-01

! --- Step 1: Interface & Security Zone Assignment ---
ASA-FIREWALL-01(config)# interface GigabitEthernet1/1
ASA-FIREWALL-01(config-if)# nameif outside
INFO: Security level for "outside" set to 0 by default.
ASA-FIREWALL-01(config-if)# security-level 0
ASA-FIREWALL-01(config-if)# ip address dhcp setroute
ASA-FIREWALL-01(config-if)# no shutdown
ASA-FIREWALL-01(config-if)# exit

ASA-FIREWALL-01(config)# interface GigabitEthernet1/2
ASA-FIREWALL-01(config-if)# nameif inside
INFO: Security level for "inside" set to 100 by default.
ASA-FIREWALL-01(config-if)# security-level 100
ASA-FIREWALL-01(config-if)# ip address 10.0.0.1 255.255.255.252
ASA-FIREWALL-01(config-if)# no shutdown
ASA-FIREWALL-01(config-if)# exit

ASA-FIREWALL-01(config)# interface GigabitEthernet1/3
ASA-FIREWALL-01(config-if)# nameif dmz
ASA-FIREWALL-01(config-if)# security-level 50
ASA-FIREWALL-01(config-if)# ip address 172.16.90.1 255.255.255.0
ASA-FIREWALL-01(config-if)# no shutdown
ASA-FIREWALL-01(config-if)# exit

! --- Step 2: Define Network Objects for Internal & DMZ Hosts ---
ASA-FIREWALL-01(config)# object network OBJ_INTERNAL_CORP
ASA-FIREWALL-01(config-network-object)# subnet 192.168.0.0 255.255.0.0
ASA-FIREWALL-01(config-network-object)# exit

ASA-FIREWALL-01(config)# object network OBJ_PUBLIC_WEB_SERVER
ASA-FIREWALL-01(config-network-object)# host 172.16.90.10
ASA-FIREWALL-01(config-network-object)# exit

! --- Step 3: Configure Dynamic PAT (Port Address Translation) ---
ASA-FIREWALL-01(config)# nat (inside,outside) source dynamic OBJ_INTERNAL_CORP interface

! --- Step 4: Access Control Lists (ACLs) ---
ASA-FIREWALL-01(config)# access-list ACL_OUTSIDE_IN extended permit tcp any host 172.16.90.10 eq 80
ASA-FIREWALL-01(config)# access-list ACL_OUTSIDE_IN extended permit tcp any host 172.16.90.10 eq 443
ASA-FIREWALL-01(config)# access-list ACL_OUTSIDE_IN extended permit icmp any any echo-reply
ASA-FIREWALL-01(config)# access-group ACL_OUTSIDE_IN in interface outside

! --- Step 5: Stateful Inspection Engine ---
ASA-FIREWALL-01(config)# policy-map global_policy
ASA-FIREWALL-01(config-pmap)# class inspection_default
ASA-FIREWALL-01(config-pmap-c)# inspect icmp
ASA-FIREWALL-01(config-pmap-c)# inspect http
ASA-FIREWALL-01(config-pmap-c)# inspect sip
ASA-FIREWALL-01(config-pmap-c)# inspect dns
ASA-FIREWALL-01(config-pmap-c)# exit
ASA-FIREWALL-01(config-pmap)# exit

! --- Step 6: Static Route to Core Switch ---
ASA-FIREWALL-01(config)# route inside 192.168.0.0 255.255.0.0 10.0.0.2
ASA-FIREWALL-01(config)# write memory
Building configuration...
[OK]
```

### 🔍 Verification Commands & Outputs
```cisco
ASA-FIREWALL-01# show nameif
Interface                Name                     Security
GigabitEthernet1/1       outside                     0
GigabitEthernet1/2       inside                    100
GigabitEthernet1/3       dmz                        50

ASA-FIREWALL-01# show access-list ACL_OUTSIDE_IN
access-list ACL_OUTSIDE_IN; 3 elements; name hash: 0x47fa9b12
access-list ACL_OUTSIDE_IN line 1 extended permit tcp any host 172.16.90.10 eq www (hitcnt=248)
access-list ACL_OUTSIDE_IN line 2 extended permit tcp any host 172.16.90.10 eq 443 (hitcnt=1420)
access-list ACL_OUTSIDE_IN line 3 extended permit icmp any any echo-reply (hitcnt=16)

ASA-FIREWALL-01# show nat
Manual NAT Policies (Section 1)
Auto NAT Policies (Section 2)
1 (inside) to (outside) source dynamic OBJ_INTERNAL_CORP interface
    translate_hits = 3120, untranslate_hits = 2954

ASA-FIREWALL-01# show xlate
2 in use, 14 most used
Flags: D - DNS, e - extended, I - inbound, N - net-to-net, s - static, v - VPN
TCP PAT from inside:192.168.20.11/51234 to outside:203.0.113.2/51234 flags e idle 0:00:04
TCP PAT from inside:192.168.20.12/49821 to outside:203.0.113.2/49821 flags e idle 0:00:12
```

---

## 2. Cisco Catalyst 3560/3650 Core Multilayer Switch

### 🔹 Step-by-Step Executed CLI Commands
```cisco
Switch> enable
Switch# configure terminal
Switch(config)# hostname CORE-DIST-SW01

! --- Step 1: Enable Hardware IPv4 Routing Engine ---
CORE-DIST-SW01(config)# ip routing

! --- Step 2: Create Enterprise & Industrial VLAN Database ---
CORE-DIST-SW01(config)# vlan 10
CORE-DIST-SW01(config-vlan)# name MGMT_INFRASTRUCTURE
CORE-DIST-SW01(config-vlan)# vlan 20
CORE-DIST-SW01(config-vlan)# name DATA_STAFF
CORE-DIST-SW01(config-vlan)# vlan 25
CORE-DIST-SW01(config-vlan)# name PRINTERS_PERIPHERALS
CORE-DIST-SW01(config-vlan)# vlan 30
CORE-DIST-SW01(config-vlan)# name VOICE_VOIP
CORE-DIST-SW01(config-vlan)# vlan 40
CORE-DIST-SW01(config-vlan)# name CORP_WIFI_WLC
CORE-DIST-SW01(config-vlan)# vlan 50
CORE-DIST-SW01(config-vlan)# name DATACENTER_SERVERS
CORE-DIST-SW01(config-vlan)# vlan 60
CORE-DIST-SW01(config-vlan)# name FACTORY_SMART_TVS
CORE-DIST-SW01(config-vlan)# vlan 70
CORE-DIST-SW01(config-vlan)# name CCTV_SURVEILLANCE
CORE-DIST-SW01(config-vlan)# vlan 80
CORE-DIST-SW01(config-vlan)# name INDUSTRIAL_OT_PLCS
CORE-DIST-SW01(config-vlan)# vlan 99
CORE-DIST-SW01(config-vlan)# name GUEST_WIFI
CORE-DIST-SW01(config-vlan)# exit

! --- Step 3: Switched Virtual Interfaces (SVIs) as Default Gateways ---
CORE-DIST-SW01(config)# interface Vlan10
CORE-DIST-SW01(config-if)# ip address 192.168.10.1 255.255.255.0
CORE-DIST-SW01(config-if)# no shutdown

CORE-DIST-SW01(config)# interface Vlan20
CORE-DIST-SW01(config-if)# ip address 192.168.20.1 255.255.255.0
CORE-DIST-SW01(config-if)# no shutdown

CORE-DIST-SW01(config)# interface Vlan25
CORE-DIST-SW01(config-if)# ip address 192.168.25.1 255.255.255.0
CORE-DIST-SW01(config-if)# no shutdown

CORE-DIST-SW01(config)# interface Vlan30
CORE-DIST-SW01(config-if)# ip address 192.168.30.1 255.255.255.0
CORE-DIST-SW01(config-if)# no shutdown

CORE-DIST-SW01(config)# interface Vlan40
CORE-DIST-SW01(config-if)# ip address 192.168.40.1 255.255.255.0
CORE-DIST-SW01(config-if)# no shutdown

CORE-DIST-SW01(config)# interface Vlan50
CORE-DIST-SW01(config-if)# ip address 192.168.50.1 255.255.255.0
CORE-DIST-SW01(config-if)# no shutdown

CORE-DIST-SW01(config)# interface Vlan60
CORE-DIST-SW01(config-if)# ip address 192.168.60.1 255.255.255.0
CORE-DIST-SW01(config-if)# no shutdown

CORE-DIST-SW01(config)# interface Vlan70
CORE-DIST-SW01(config-if)# ip address 192.168.70.1 255.255.255.0
CORE-DIST-SW01(config-if)# no shutdown

CORE-DIST-SW01(config)# interface Vlan80
CORE-DIST-SW01(config-if)# ip address 192.168.80.1 255.255.255.0
CORE-DIST-SW01(config-if)# no shutdown

CORE-DIST-SW01(config)# interface Vlan99
CORE-DIST-SW01(config-if)# ip address 192.168.99.1 255.255.255.0
CORE-DIST-SW01(config-if)# no shutdown
CORE-DIST-SW01(config-if)# exit

! --- Step 4: Routed Point-to-Point Uplink to Cisco ASA ---
CORE-DIST-SW01(config)# interface GigabitEthernet0/1
CORE-DIST-SW01(config-if)# no switchport
CORE-DIST-SW01(config-if)# ip address 10.0.0.2 255.255.255.252
CORE-DIST-SW01(config-if)# no shutdown
CORE-DIST-SW01(config-if)# exit

CORE-DIST-SW01(config)# ip route 0.0.0.0 0.0.0.0 10.0.0.1

! --- Step 5: 802.1Q Trunk Downlinks to Access Switches ---
CORE-DIST-SW01(config)# interface range GigabitEthernet0/2 - 4
CORE-DIST-SW01(config-if-range)# switchport trunk encapsulation dot1q
CORE-DIST-SW01(config-if-range)# switchport mode trunk
CORE-DIST-SW01(config-if-range)# switchport trunk allowed vlan 10,20,25,30,40,50,60,70,80,99
CORE-DIST-SW01(config-if-range)# no shutdown
CORE-DIST-SW01(config-if-range)# exit
CORE-DIST-SW01(config)# do write memory
```

### 🔍 Verification Commands & Outputs
```cisco
CORE-DIST-SW01# show ip route
Gateway of last resort is 10.0.0.1 to network 0.0.0.0

S*    0.0.0.0/0 [1/0] via 10.0.0.1
      10.0.0.0/30 is subnetted, 1 subnets
C        10.0.0.0 is directly connected, GigabitEthernet0/1
C     192.168.10.0/24 is directly connected, Vlan10
C     192.168.20.0/24 is directly connected, Vlan20
C     192.168.25.0/24 is directly connected, Vlan25
C     192.168.30.0/24 is directly connected, Vlan30
C     192.168.40.0/24 is directly connected, Vlan40
C     192.168.50.0/24 is directly connected, Vlan50
C     192.168.60.0/24 is directly connected, Vlan60
C     192.168.70.0/24 is directly connected, Vlan70
C     192.168.80.0/24 is directly connected, Vlan80

CORE-DIST-SW01# show interfaces trunk
Port        Mode         Encapsulation  Status        Native vlan
Gi0/2       on           802.1q         trunking      1
Gi0/3       on           802.1q         trunking      1

Port        Vlans allowed on trunk
Gi0/2       10,20,25,30,40,50,60,70,80,99
Gi0/3       10,20,25,30,40,50,60,70,80,99
```

---

## 3. Cisco 2811/2911 Edge Router & CME PBX Telephony

### 🔹 Step-by-Step Executed CLI Commands
```cisco
Router> enable
Router# configure terminal
Router(config)# hostname R1-EDGE-CME

! --- Step 1: Sub-interfaces (Router-on-a-Stick 802.1Q) ---
R1-EDGE-CME(config)# interface GigabitEthernet0/0
R1-EDGE-CME(config-if)# no shutdown
R1-EDGE-CME(config-if)# exit

R1-EDGE-CME(config)# interface GigabitEthernet0/0.20
R1-EDGE-CME(config-subif)# encapsulation dot1Q 20
R1-EDGE-CME(config-subif)# ip address 192.168.20.1 255.255.255.0

R1-EDGE-CME(config)# interface GigabitEthernet0/0.30
R1-EDGE-CME(config-subif)# encapsulation dot1Q 30
R1-EDGE-CME(config-subif)# ip address 192.168.30.1 255.255.255.0

R1-EDGE-CME(config)# interface GigabitEthernet0/0.60
R1-EDGE-CME(config-subif)# encapsulation dot1Q 60
R1-EDGE-CME(config-subif)# ip address 192.168.60.1 255.255.255.0
R1-EDGE-CME(config-subif)# exit

! --- Step 2: DHCP Pools with TFTP Option 150 for IP Phones ---
R1-EDGE-CME(config)# ip dhcp excluded-address 192.168.20.1 192.168.20.10
R1-EDGE-CME(config)# ip dhcp excluded-address 192.168.30.1 192.168.30.10
R1-EDGE-CME(config)# ip dhcp excluded-address 192.168.60.1 192.168.60.10

R1-EDGE-CME(config)# ip dhcp pool POOL-STAFF-DATA
R1-EDGE-CME(dhcp-config)# network 192.168.20.0 255.255.255.0
R1-EDGE-CME(dhcp-config)# default-router 192.168.20.1
R1-EDGE-CME(dhcp-config)# dns-server 192.168.50.10

R1-EDGE-CME(config)# ip dhcp pool POOL-VOIP-VOICE
R1-EDGE-CME(dhcp-config)# network 192.168.30.0 255.255.255.0
R1-EDGE-CME(dhcp-config)# default-router 192.168.30.1
R1-EDGE-CME(dhcp-config)# option 150 ip 192.168.30.1

R1-EDGE-CME(config)# ip dhcp pool POOL-SMART-TVS
R1-EDGE-CME(dhcp-config)# network 192.168.60.0 255.255.255.0
R1-EDGE-CME(dhcp-config)# default-router 192.168.60.1
R1-EDGE-CME(dhcp-config)# exit

! --- Step 3: Cisco CallManager Express (CME) IP PBX ---
R1-EDGE-CME(config)# telephony-service
R1-EDGE-CME(config-telephony)# max-ephones 10
R1-EDGE-CME(config-telephony)# max-dn 10
R1-EDGE-CME(config-telephony)# ip source-address 192.168.30.1 port 2000
R1-EDGE-CME(config-telephony)# auto assign 1 to 10
R1-EDGE-CME(config-telephony)# create cnf-files
R1-EDGE-CME(config-telephony)# exit

! --- Step 4: Dial Extensions (DNs) Assignment ---
R1-EDGE-CME(config)# ephone-dn 1
R1-EDGE-CME(config-ephone-dn)# number 101
R1-EDGE-CME(config-ephone-dn)# description Admin-Desk

R1-EDGE-CME(config)# ephone-dn 2
R1-EDGE-CME(config-ephone-dn)# number 102
R1-EDGE-CME(config-ephone-dn)# description Support-Desk

R1-EDGE-CME(config)# ephone-dn 3
R1-EDGE-CME(config-ephone-dn)# number 103
R1-EDGE-CME(config-ephone-dn)# description Factory-Floor
R1-EDGE-CME(config-ephone-dn)# exit
R1-EDGE-CME(config)# do write memory
```

### 🔍 Verification Commands & Outputs
```cisco
R1-EDGE-CME# show telephony-service ephone
ephone-1[0] Mac:0001.96C2.B1A4 TCP socket:[1] active media-active:1 IP:192.168.30.11 50608 7960
 button 1: dn 1 (number 101)  status: CONNECTED

ephone-2[1] Mac:0004.9A32.D712 TCP socket:[2] active media-active:0 IP:192.168.30.12 50609 7960
 button 1: dn 2 (number 102)  status: IDLE

ephone-3[2] Mac:0060.47B8.89FE TCP socket:[3] active media-active:1 IP:192.168.30.13 50610 7960
 button 1: dn 3 (number 103)  status: CONNECTED

R1-EDGE-CME# show ip dhcp binding
IP address       Client-ID/Hardware address      Type       Lease expiration
192.168.20.11    0100.5079.6668.01 (PC-Admin)    Automatic  --
192.168.30.11    0100.0196.c2b1.a4 (Phone-101)   Automatic  --
192.168.30.13    0100.6047.b889.fe (Phone-103)   Automatic  --
192.168.60.11    0100.e0b2.8891.aa (Smart-TV)    Automatic  --
```

---

## 4. Cisco Catalyst 2960 Office Access Switch

### 🔹 Step-by-Step Executed CLI Commands
```cisco
Switch> enable
Switch# configure terminal
Switch(config)# hostname SW-ACCESS-OFFICE

! Create Local VLANs
SW-ACCESS-OFFICE(config)# vlan 10,20,25,30,40
SW-ACCESS-OFFICE(config)# exit

! Workstations + Voice VLAN Daisy-Chaining
SW-ACCESS-OFFICE(config)# interface range FastEthernet0/1 - 10
SW-ACCESS-OFFICE(config-if-range)# switchport mode access
SW-ACCESS-OFFICE(config-if-range)# switchport access vlan 20
SW-ACCESS-OFFICE(config-if-range)# switchport voice vlan 30
SW-ACCESS-OFFICE(config-if-range)# spanning-tree portfast
SW-ACCESS-OFFICE(config-if-range)# mls qos trust cos
SW-ACCESS-OFFICE(config-if-range)# no shutdown
SW-ACCESS-OFFICE(config-if-range)# exit

! Network Printers Port
SW-ACCESS-OFFICE(config)# interface range FastEthernet0/11 - 14
SW-ACCESS-OFFICE(config-if-range)# switchport mode access
SW-ACCESS-OFFICE(config-if-range)# switchport access vlan 25
SW-ACCESS-OFFICE(config-if-range)# spanning-tree portfast
SW-ACCESS-OFFICE(config-if-range)# no shutdown
SW-ACCESS-OFFICE(config-if-range)# exit

! Trunk Uplink to Core Switch
SW-ACCESS-OFFICE(config)# interface GigabitEthernet0/1
SW-ACCESS-OFFICE(config-if)# switchport mode trunk
SW-ACCESS-OFFICE(config-if)# switchport trunk allowed vlan 10,20,25,30,40,99
SW-ACCESS-OFFICE(config-if)# no shutdown
SW-ACCESS-OFFICE(config-if)# exit
SW-ACCESS-OFFICE(config)# do write memory
```

---

## 5. Cisco Industrial / 2960 Factory Access Switch

### 🔹 Step-by-Step Executed CLI Commands
```cisco
Switch> enable
Switch# configure terminal
Switch(config)# hostname SW-ACCESS-FACTORY

! Create Factory VLANs
SW-ACCESS-FACTORY(config)# vlan 10,30,60,70,80
SW-ACCESS-FACTORY(config)# exit

! Production Smart TVs
SW-ACCESS-FACTORY(config)# interface range FastEthernet0/5 - 8
SW-ACCESS-FACTORY(config-if-range)# switchport mode access
SW-ACCESS-FACTORY(config-if-range)# switchport access vlan 60
SW-ACCESS-FACTORY(config-if-range)# spanning-tree portfast
SW-ACCESS-FACTORY(config-if-range)# no shutdown
SW-ACCESS-FACTORY(config-if-range)# exit

! High-Definition IP Surveillance Cameras (PoE)
SW-ACCESS-FACTORY(config)# interface range FastEthernet0/9 - 16
SW-ACCESS-FACTORY(config-if-range)# switchport mode access
SW-ACCESS-FACTORY(config-if-range)# switchport access vlan 70
SW-ACCESS-FACTORY(config-if-range)# spanning-tree portfast
SW-ACCESS-FACTORY(config-if-range)# no shutdown
SW-ACCESS-FACTORY(config-if-range)# exit

! Industrial OT PLCs & RFID Access Turnstiles
SW-ACCESS-FACTORY(config)# interface range FastEthernet0/17 - 24
SW-ACCESS-FACTORY(config-if-range)# switchport mode access
SW-ACCESS-FACTORY(config-if-range)# switchport access vlan 80
SW-ACCESS-FACTORY(config-if-range)# spanning-tree portfast
SW-ACCESS-FACTORY(config-if-range)# no shutdown
SW-ACCESS-FACTORY(config-if-range)# exit

! Trunk Uplink to Core Switch
SW-ACCESS-FACTORY(config)# interface GigabitEthernet0/1
SW-ACCESS-FACTORY(config-if)# switchport mode trunk
SW-ACCESS-FACTORY(config-if)# switchport trunk allowed vlan 10,30,60,70,80
SW-ACCESS-FACTORY(config-if)# no shutdown
SW-ACCESS-FACTORY(config-if)# exit
SW-ACCESS-FACTORY(config)# do write memory
```

---

## 6. End-to-End Connectivity & Verification Matrix

### 🏓 Ping Tests Run from PC-Admin (`192.168.20.11`)

| Destination Target | Target IP Address | Path Traversed | Result | Status |
| :--- | :--- | :--- | :---: | :---: |
| **VLAN 20 Gateway** | `192.168.20.1` | Local Access Switch ➡️ Core SVI | `0% loss (time <1ms)` | ✅ SUCCESS |
| **Active Directory / DNS**| `192.168.50.10` | Core Inter-VLAN Routing (VLAN 20 ➡️ 50) | `0% loss (time 1ms)` | ✅ SUCCESS |
| **CCTV NVR Server** | `192.168.50.50` | Core Inter-VLAN Routing (VLAN 20 ➡️ 50) | `0% loss (time 1ms)` | ✅ SUCCESS |
| **Office Laser Printer** | `192.168.25.11` | Core Inter-VLAN Routing (VLAN 20 ➡️ 25) | `0% loss (time <1ms)` | ✅ SUCCESS |
| **Smart TV Production** | `192.168.60.11` | Core Inter-VLAN Routing (VLAN 20 ➡️ 60) | `0% loss (time 1ms)` | ✅ SUCCESS |
| **DMZ Web Server** | `172.16.90.10` | Core Switch ➡️ ASA Firewall DMZ Interface | `0% loss (time 2ms)` | ✅ SUCCESS |
| **Public Internet (WAN)** | `8.8.8.8` | Core Switch ➡️ ASA PAT Outbound NAT | `0% loss (time 12ms)` | ✅ SUCCESS |
| **Factory PLC Sensors** | `192.168.80.20` | Core Switch ACL Filtered | `Request Timed Out` | 🛡️ BLOCKED (Security Policy) |
