# 🏢 Documentazione Generale del Progetto di Rete Aziendale e Smart Factory
## Progetto CCNA / Enterprise Campus & Industrial IoT

Progetto di rete integrato basato sulla segmentazione gerarchica, isolamento di sicurezza mediante VLAN (Virtual Local Area Network) e garanzia della qualità del servizio (QoS) per flussi multimediali convergenti (voce VoIP, video CCTV ad alta definizione, telemetria industriale, cruscotti Smart TV e server applicativi).

---

## 1. Dispositivi e Tecnologie Adottate

| Dispositivo | Ruolo nell'Infrastruttura | Tecnologia e Protocolli Implementati |
| :--- | :--- | :--- |
| **Next-Gen Firewall (Cisco ASA 5506-X)** | Sicurezza perimetrale, zonizzazione Outside/Inside/DMZ, NAT/PAT | Stateful Packet Inspection, Dynamic PAT, Access Control Lists (ACL) |
| **Switch Core / Distribuzione Layer 3** | Routing ad altissima velocità tra le VLAN (Inter-VLAN Routing) | Cisco Catalyst 3560/3650, SVIs, Routed Uplink Point-to-Point |
| **Switch di Accesso PoE (Uffici e Fabbrica)**| Distribuzione porte, alimentazione remota PoE (802.3at/af) | Cisco Catalyst 2960-24PC-L / Industrial Ethernet IE-2000 |
| **Telefonia VoIP Cisco CME** | Comunicazioni vocali aziendali e di stabilimento con QoS | Cisco CallManager Express (CME), Cisco IP Phone 7960, CoS 5 (Voice VLAN) |
| **Videosorveglianza IP (CCTV)** | Registrazione continua 24/7 dei reparti e flussi di sicurezza | Telecamere IP HD PoE, Server NVR dedicato, VLAN 70 isolata |
| **Smart TV e Display Industriali** | Monitoraggio in tempo reale dei parametri di linea e cruscotti OEE | Smart Display / Thin Clients, DHCP dedicato (VLAN 60) |
| **Controllo Accessi e Sensori OT** | Sicurezza fisica, tornelli biometrici e telemetria SCADA | Terminali RFID, sensori di temperatura e PLC industriali (VLAN 80) |
| **Server Farm Enterprise** | Servizi centralizzati di dominio, file, ERP e storage | Active Directory, DNS, ERP aziendale e NVR Storage (VLAN 50) |
| **Infrastruttura Wireless** | Connettività sicura per laptop aziendali e ospiti | Cisco WLC (Wireless LAN Controller), Access Point Dual-Band (VLAN 40/99) |

---

## 2. Matrice di Indirizzamento e Assegnazione VLAN

| ID VLAN | Nome Segmento | Rete Subnet | Gateway Predefinito | Maschera Subnet | Destinazione d'Uso |
| :---: | :--- | :--- | :--- | :--- | :--- |
| **10** | `MGMT_INFRA` | `192.168.10.0/24` | `192.168.10.1` | `255.255.255.0` | Gestione in-band degli apparati (SSHv2 / SNMPv3) |
| **20** | `DATA_STAFF` | `192.168.20.0/24` | `192.168.20.1` | `255.255.255.0` | Postazioni di lavoro PC del personale (DHCP) |
| **25** | `PRINTERS` | `192.168.25.0/24` | `192.168.25.1` | `255.255.255.0` | Stampanti laser e scanner multifunzione di reparto |
| **30** | `VOICE_VOIP` | `192.168.30.0/24` | `192.168.30.1` | `255.255.255.0` | Telefoni IP Cisco 7960 con priorità audio CoS 5 |
| **40** | `CORP_WIFI` | `192.168.40.0/24` | `192.168.40.1` | `255.255.255.0` | Wi-Fi aziendale gestito da WLC (WPA3-Enterprise) |
| **50** | `SERVERS_DC` | `192.168.50.0/24` | `192.168.50.1` | `255.255.255.0` | Server farm: Active Directory, ERP, NVR CCTV |
| **60** | `SMART_TVS` | `192.168.60.0/24` | `192.168.60.1` | `255.255.255.0` | Smart TV industriali per cruscotti KPI di linea |
| **70** | `CCTV_CAMERAS`| `192.168.70.0/24`| `192.168.70.1` | `255.255.255.0` | Telecamere di videosorveglianza PoE isolate |
| **80** | `FACTORY_OT` | `192.168.80.0/24` | `192.168.80.1` | `255.255.255.0` | PLC industriali, sensori d'impianto, varchi RFID |
| **90** | `DMZ_PUBLIC` | `172.16.90.0/24` | `172.16.90.1` | `255.255.255.0` | Web Server pubblico / Portale clienti (`.90.10`) |
| **99** | `GUEST_WIFI` | `192.168.99.0/24` | `192.168.99.1` | `255.255.255.0` | Wi-Fi ospiti con captive portal (accesso solo WAN) |

---

## 3. Tecnologia VoIP Daisy-Chaining (Cavo Unico Scrivania)

I telefoni IP Cisco 7960 integrano al loro interno un mini-switch a 3 porte che permette di collegare sia il computer che il telefono utilizzando una sola presa di rete a muro:

1. Un cavo patch collega lo switch di accesso PoE alla porta **SW** sul retro del telefono IP.
2. Un secondo cavo patch collega la porta **PC** del telefono alla scheda di rete del computer.
3. Lo switch Cisco riconosce automaticamente il traffico e applica il tagging:
   - I pacchetti voce vengono instradati sulla **VLAN 30 (Voice VLAN)** con marcatura 802.1Q e alta priorità CoS 5.
   - I pacchetti dati del computer transitano sulla **VLAN 20 (Data VLAN)** in modalità Untagged nativa, garantendo isolamento di sicurezza totale senza conflitti.
