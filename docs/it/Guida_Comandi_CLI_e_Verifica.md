# 💻 Guida ai Comandi CLI ed Esecuzione dei Log di Verifica
### Documentazione Completa dei Comandi Cisco per Router, Firewall e Switch con Output di Verifica

Questa guida fornisce la trascrizione dettagliata riga per riga di tutti i comandi CLI eseguiti sui dispositivi di rete del progetto **SicuraNet Core: Rete Aziendale e Smart Factory CCNA**, con spiegazioni tecniche in italiano e output dei comandi di diagnostica (`show`, `debug`, `ping`).

---

## 📑 Indice dei Contenuti
1. [Firewall Perimetrale (Cisco ASA 5506-X)](#1-firewall-perimetrale-cisco-asa-5506-x)
2. [Switch Core di Distribuzione Layer 3 (Cisco Catalyst 3560/3650)](#2-switch-core-di-distribuzione-layer-3-cisco-catalyst-35603650)
3. [Router di Bordo & PBX VoIP Telephony (Cisco 2811/2911 CME)](#3-router-di-bordo--pbx-voip-telephony-cisco-28112911-cme)
4. [Switch di Accesso Uffici (Cisco Catalyst 2960 Office)](#4-switch-di-accesso-uffici-cisco-catalyst-2960-office)
5. [Switch Industriale Fabbrica (Cisco IE-2000 / Catalyst 2960 Factory)](#5-switch-industriale-fabbrica-cisco-ie-2000--catalyst-2960-factory)
6. [Tabella di Verifica e Diagnostica Globale (Troubleshooting)](#6-tabella-di-verifica-e-diagnostica-globale-troubleshooting)

---

## 1. Firewall Perimetrale (Cisco ASA 5506-X)

Il firewall perimetrale costituisce la prima linea di difesa, segmentando la rete esterna (Internet/Outside), la zona demilitarizzata (DMZ) e la rete aziendale interna (Inside).

### A) Sequenza di Configurazione CLI:
```cisco
ciscoasa> enable
ciscoasa# configure terminal
ciscoasa(config)# hostname ASA-FIREWALL-01

! --- 1. Configurazione Interfacce e Livelli di Sicurezza (Security Levels) ---
! Interfaccia esterna verso il provider ISP (Livello di sicurezza 0 = Non affidabile)
ASA-FIREWALL-01(config)# interface GigabitEthernet1/1
ASA-FIREWALL-01(config-if)# nameif outside
INFO: Security level for "outside" set to 0 by default.
ASA-FIREWALL-01(config-if)# security-level 0
ASA-FIREWALL-01(config-if)# ip address dhcp setroute
ASA-FIREWALL-01(config-if)# no shutdown
ASA-FIREWALL-01(config-if)# exit

! Interfaccia interna verso lo Switch Core (Livello 100 = Massima affidabilità)
ASA-FIREWALL-01(config)# interface GigabitEthernet1/2
ASA-FIREWALL-01(config-if)# nameif inside
INFO: Security level for "inside" set to 100 by default.
ASA-FIREWALL-01(config-if)# security-level 100
ASA-FIREWALL-01(config-if)# ip address 10.0.0.1 255.255.255.252
ASA-FIREWALL-01(config-if)# no shutdown
ASA-FIREWALL-01(config-if)# exit

! Interfaccia perimetrale DMZ per server pubblici (Livello 50 = Semi-affidabile)
ASA-FIREWALL-01(config)# interface GigabitEthernet1/3
ASA-FIREWALL-01(config-if)# nameif dmz
ASA-FIREWALL-01(config-if)# security-level 50
ASA-FIREWALL-01(config-if)# ip address 172.16.90.1 255.255.255.0
ASA-FIREWALL-01(config-if)# no shutdown
ASA-FIREWALL-01(config-if)# exit

! --- 2. Definizione Network Objects per Host e Subnet ---
ASA-FIREWALL-01(config)# object network OBJ_INTERNAL_CORP
ASA-FIREWALL-01(config-network-object)# subnet 192.168.0.0 255.255.0.0
ASA-FIREWALL-01(config-network-object)# exit

ASA-FIREWALL-01(config)# object network OBJ_PUBLIC_WEB_SERVER
ASA-FIREWALL-01(config-network-object)# host 172.16.90.10
ASA-FIREWALL-01(config-network-object)# exit

! --- 3. Configurazione Dynamic PAT per Uscita Internet ---
ASA-FIREWALL-01(config)# nat (inside,outside) source dynamic OBJ_INTERNAL_CORP interface

! --- 4. Regole di Controllo Accessi (ACL) ---
! Consente solo traffico HTTP/HTTPS verso il Web Server in DMZ
ASA-FIREWALL-01(config)# access-list ACL_OUTSIDE_IN extended permit tcp any host 172.16.90.10 eq 80
ASA-FIREWALL-01(config)# access-list ACL_OUTSIDE_IN extended permit tcp any host 172.16.90.10 eq 443
ASA-FIREWALL-01(config)# access-list ACL_OUTSIDE_IN extended permit icmp any any echo-reply
ASA-FIREWALL-01(config)# access-group ACL_OUTSIDE_IN in interface outside

! --- 5. Ispezione Dinamica dei Pacchetti (Stateful Inspection) ---
ASA-FIREWALL-01(config)# policy-map global_policy
ASA-FIREWALL-01(config-pmap)# class inspection_default
ASA-FIREWALL-01(config-pmap-c)# inspect icmp
ASA-FIREWALL-01(config-pmap-c)# inspect http
ASA-FIREWALL-01(config-pmap-c)# inspect sip
ASA-FIREWALL-01(config-pmap-c)# inspect dns
ASA-FIREWALL-01(config-pmap-c)# exit
ASA-FIREWALL-01(config-pmap)# exit

! --- 6. Routing Statico verso lo Switch Core Layer 3 ---
ASA-FIREWALL-01(config)# route inside 192.168.0.0 255.255.0.0 10.0.0.2
ASA-FIREWALL-01(config)# write memory
Building configuration...
[OK]
```

### B) Output dei Comandi di Verifica sul Firewall:
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

## 2. Switch Core di Distribuzione Layer 3 (Cisco Catalyst 3560/3650)

Gestisce tutto il routing tra le diverse VLAN aziendali a livello hardware (Wire-speed Inter-VLAN Routing) mediante le interfacce virtuali di switch (SVI).

### A) Sequenza di Configurazione CLI:
```cisco
Switch> enable
Switch# configure terminal
Switch(config)# hostname CORE-DIST-SW01

! --- 1. Abilitazione Motore di Routing IPv4 ---
CORE-DIST-SW01(config)# ip routing

! --- 2. Creazione del Database VLAN Aziendale e Industriale ---
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

! --- 3. Creazione Gateways Predefiniti (SVIs) ---
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

! --- 4. Porta Routed Point-to-Point verso il Firewall ASA ---
CORE-DIST-SW01(config)# interface GigabitEthernet0/1
CORE-DIST-SW01(config-if)# no switchport
CORE-DIST-SW01(config-if)# ip address 10.0.0.2 255.255.255.252
CORE-DIST-SW01(config-if)# no shutdown
CORE-DIST-SW01(config-if)# exit

CORE-DIST-SW01(config)# ip route 0.0.0.0 0.0.0.0 10.0.0.1

! --- 5. Dorsali 802.1Q Trunk verso gli Switch di Accesso ---
CORE-DIST-SW01(config)# interface range GigabitEthernet0/2 - 4
CORE-DIST-SW01(config-if-range)# switchport trunk encapsulation dot1q
CORE-DIST-SW01(config-if-range)# switchport mode trunk
CORE-DIST-SW01(config-if-range)# switchport trunk allowed vlan 10,20,25,30,40,50,60,70,80,99
CORE-DIST-SW01(config-if-range)# no shutdown
CORE-DIST-SW01(config-if-range)# exit
CORE-DIST-SW01(config)# do write memory
```

### B) Output dei Comandi di Verifica sullo Switch Core:
```cisco
CORE-DIST-SW01# show ip route
Gateway of last resort is 10.0.0.1 to network 0.0.0.0

S*    0.0.0.0/0 [1/0] via 10.0.0.1
C     10.0.0.0/30 is directly connected, GigabitEthernet0/1
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

## 3. Router di Bordo & PBX VoIP Telephony (Cisco 2811/2911 CME)

Fornisce le funzioni di centralino VoIP IP-PBX (Cisco CallManager Express) e distribuisce gli indirizzi IP tramite server DHCP multi-pool con l'opzione `Option 150`.

### A) Sequenza di Configurazione CLI:
```cisco
Router> enable
Router# configure terminal
Router(config)# hostname R1-EDGE-CME

! --- 1. Sotto-interfacce 802.1Q (Router-on-a-Stick) ---
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

! --- 2. Pool DHCP Multipli con Option 150 ---
R1-EDGE-CME(config)# ip dhcp excluded-address 192.168.20.1 192.168.20.10
R1-EDGE-CME(config)# ip dhcp excluded-address 192.168.30.1 192.168.30.10
R1-EDGE-CME(config)# ip dhcp excluded-address 192.168.60.1 192.168.60.10

! Pool PC Uffici
R1-EDGE-CME(config)# ip dhcp pool POOL-STAFF-DATA
R1-EDGE-CME(dhcp-config)# network 192.168.20.0 255.255.255.0
R1-EDGE-CME(dhcp-config)# default-router 192.168.20.1
R1-EDGE-CME(dhcp-config)# dns-server 192.168.50.10

! Pool Telefonia VoIP (Option 150 definisce l'IP del TFTP server per i file di config)
R1-EDGE-CME(config)# ip dhcp pool POOL-VOIP-VOICE
R1-EDGE-CME(dhcp-config)# network 192.168.30.0 255.255.255.0
R1-EDGE-CME(dhcp-config)# default-router 192.168.30.1
R1-EDGE-CME(dhcp-config)# option 150 ip 192.168.30.1

! Pool Smart TV Fabbrica
R1-EDGE-CME(config)# ip dhcp pool POOL-SMART-TVS
R1-EDGE-CME(dhcp-config)# network 192.168.60.0 255.255.255.0
R1-EDGE-CME(dhcp-config)# default-router 192.168.60.1
R1-EDGE-CME(dhcp-config)# exit

! --- 3. Servizio Telefonico Cisco CME ---
R1-EDGE-CME(config)# telephony-service
R1-EDGE-CME(config-telephony)# max-ephones 10
R1-EDGE-CME(config-telephony)# max-dn 10
R1-EDGE-CME(config-telephony)# ip source-address 192.168.30.1 port 2000
R1-EDGE-CME(config-telephony)# auto assign 1 to 10
R1-EDGE-CME(config-telephony)# create cnf-files
R1-EDGE-CME(config-telephony)# exit

! --- 4. Assegnazione Interni Telefonici (DNs) ---
R1-EDGE-CME(config)# ephone-dn 1
R1-EDGE-CME(config-ephone-dn)# number 101
R1-EDGE-CME(config-ephone-dn)# description Ufficio-Direzione

R1-EDGE-CME(config)# ephone-dn 2
R1-EDGE-CME(config-ephone-dn)# number 102
R1-EDGE-CME(config-ephone-dn)# description Supporto-Tecnico

R1-EDGE-CME(config)# ephone-dn 3
R1-EDGE-CME(config-ephone-dn)# number 103
R1-EDGE-CME(config-ephone-dn)# description Reparto-Produzione
R1-EDGE-CME(config-ephone-dn)# exit
R1-EDGE-CME(config)# do write memory
```

### B) Output dei Comandi di Verifica sul Router CME:
```cisco
R1-EDGE-CME# show telephony-service ephone
ephone-1[0] Mac:0001.96C2.B1A4 TCP socket:[1] active media-active:1 IP:192.168.30.11 50608 7960
 button 1: dn 1 (number 101)  status: CONNECTED

R1-EDGE-CME# show ip dhcp binding
IP address       Client-ID/Hardware address      Type
192.168.20.11    0100.5079.6668.01 (PC-Admin)    Automatic
192.168.30.11    0100.0196.c2b1.a4 (Phone-101)   Automatic
192.168.30.13    0100.6047.b889.fe (Phone-103)   Automatic
192.168.60.11    0100.e0b2.8891.aa (Smart-TV)    Automatic
```

---

## 4. Switch di Accesso Uffici (Cisco Catalyst 2960 Office)

```cisco
Switch> enable
Switch# configure terminal
Switch(config)# hostname SW-ACCESS-OFFICE

SW-ACCESS-OFFICE(config)# vlan 10,20,25,30,40
SW-ACCESS-OFFICE(config)# exit

! Daisy-chaining PC e Telefono IP su singolo cavo a muro
SW-ACCESS-OFFICE(config)# interface range FastEthernet0/1 - 10
SW-ACCESS-OFFICE(config-if-range)# switchport mode access
SW-ACCESS-OFFICE(config-if-range)# switchport access vlan 20
SW-ACCESS-OFFICE(config-if-range)# switchport voice vlan 30
SW-ACCESS-OFFICE(config-if-range)# spanning-tree portfast
SW-ACCESS-OFFICE(config-if-range)# mls qos trust cos
SW-ACCESS-OFFICE(config-if-range)# no shutdown
SW-ACCESS-OFFICE(config-if-range)# exit

! Porte Stampanti di Rete Multifunzione
SW-ACCESS-OFFICE(config)# interface range FastEthernet0/11 - 14
SW-ACCESS-OFFICE(config-if-range)# switchport mode access
SW-ACCESS-OFFICE(config-if-range)# switchport access vlan 25
SW-ACCESS-OFFICE(config-if-range)# spanning-tree portfast
SW-ACCESS-OFFICE(config-if-range)# no shutdown
SW-ACCESS-OFFICE(config-if-range)# exit

! Uplink Dorsale Trunk verso lo Switch Core
SW-ACCESS-OFFICE(config)# interface GigabitEthernet0/1
SW-ACCESS-OFFICE(config-if)# switchport mode trunk
SW-ACCESS-OFFICE(config-if)# switchport trunk allowed vlan 10,20,25,30,40,99
SW-ACCESS-OFFICE(config-if)# no shutdown
SW-ACCESS-OFFICE(config-if)# exit
SW-ACCESS-OFFICE(config)# do write memory
```

---

## 5. Switch Industriale Fabbrica (Cisco IE-2000 / Catalyst 2960 Factory)

```cisco
Switch> enable
Switch# configure terminal
Switch(config)# hostname SW-ACCESS-FACTORY

SW-ACCESS-FACTORY(config)# vlan 10,30,60,70,80
SW-ACCESS-FACTORY(config)# exit

! Smart TV Industriali per Cruscotti OEE e Produzione
SW-ACCESS-FACTORY(config)# interface range FastEthernet0/5 - 8
SW-ACCESS-FACTORY(config-if-range)# switchport mode access
SW-ACCESS-FACTORY(config-if-range)# switchport access vlan 60
SW-ACCESS-FACTORY(config-if-range)# spanning-tree portfast
SW-ACCESS-FACTORY(config-if-range)# no shutdown
SW-ACCESS-FACTORY(config-if-range)# exit

! Telecamere di Videosorveglianza IP CCTV HD (Alimentate PoE)
SW-ACCESS-FACTORY(config)# interface range FastEthernet0/9 - 16
SW-ACCESS-FACTORY(config-if-range)# switchport mode access
SW-ACCESS-FACTORY(config-if-range)# switchport access vlan 70
SW-ACCESS-FACTORY(config-if-range)# spanning-tree portfast
SW-ACCESS-FACTORY(config-if-range)# no shutdown
SW-ACCESS-FACTORY(config-if-range)# exit

! Sensori Industriali OT e Tornelli Biometrici RFID
SW-ACCESS-FACTORY(config)# interface range FastEthernet0/17 - 24
SW-ACCESS-FACTORY(config-if-range)# switchport mode access
SW-ACCESS-FACTORY(config-if-range)# switchport access vlan 80
SW-ACCESS-FACTORY(config-if-range)# spanning-tree portfast
SW-ACCESS-FACTORY(config-if-range)# no shutdown
SW-ACCESS-FACTORY(config-if-range)# exit

! Uplink Dorsale Trunk verso lo Switch Core
SW-ACCESS-FACTORY(config)# interface GigabitEthernet0/1
SW-ACCESS-FACTORY(config-if)# switchport mode trunk
SW-ACCESS-FACTORY(config-if)# switchport trunk allowed vlan 10,30,60,70,80
SW-ACCESS-FACTORY(config-if)# no shutdown
SW-ACCESS-FACTORY(config-if)# exit
SW-ACCESS-FACTORY(config)# do write memory
```

---

## 6. Tabella di Verifica e Diagnostica Globale (Troubleshooting)

| Dispositivo | Comando di Diagnostica | Esito Atteso e Spiegazione Tecnica |
| :--- | :--- | :--- |
| **Cisco ASA** | `show nameif` | Verifica delle zone perimetrali e livelli di sicurezza (`outside=0`, `inside=100`, `dmz=50`). |
| **Cisco ASA** | `show access-list` | Controllo dei contatori di matching (`hitcnt`) sulle regole autorizzate HTTP/HTTPS. |
| **Cisco ASA** | `show xlate` | Mostra le sessioni attive tradotte dal Dynamic PAT per gli host che navigano su Internet. |
| **Core Switch** | `show ip route` | Visualizzazione della tabella con rotta di default `0.0.0.0/0` e delle 10 rotte connesse per le SVIs. |
| **Core Switch** | `show interfaces trunk` | Verifica delle porte dorsali 802.1Q e delle VLAN consentite in transito. |
| **CME Router** | `show telephony-service ephone` | Stato di registrazione e attività dei terminali telefonici IP (`CONNECTED` o `IDLE`). |
| **CME Router** | `show ip dhcp binding` | Elenco degli indirizzi IP rilasciati dinamicamente ai dispositivi client. |
| **Workstation** | `ping 192.168.50.50` | Verifica raggiungibilità ad alta velocità del server NVR tramite routing Inter-VLAN. |
| **Workstation** | `ping 172.16.90.10` | Verifica raggiungibilità del Web Server in DMZ attraverso l'ispezione dell'ASA. |
