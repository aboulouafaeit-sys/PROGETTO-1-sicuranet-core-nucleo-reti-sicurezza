# 🛠️ Guida alla Costruzione del Progetto in Cisco Packet Tracer Passo-Passo

Questa guida illustra dettagliatamente la procedura per assemblare, cablare e configurare l'intero progetto su **Cisco Packet Tracer**, attivando la telefonia VoIP reale tra uffici e stabilimento, le Smart TV industriali, la videosorveglianza CCTV e l'infrastruttura perimetrale protetta da firewall.

---

## 1. Dispositivi Richiesti dalla Barra degli Strumenti di Packet Tracer

| Categoria | Modello Dispositivo | Q.tà | Ruolo e Funzione |
| :--- | :--- | :---: | :--- |
| **Routers** | **2811** o **2911** | 1 | Router di bordo + Server DHCP + Centralino VoIP CME |
| **Firewall** | **ASA 5506-X** | 1 | Sicurezza perimetrale, zonizzazione Outside/Inside/DMZ, NAT/PAT |
| **Core Switch** | **3560-24PS** o **3650-24PS** | 1 | Switch Layer 3 per routing Inter-VLAN e dorsali ad alta velocità |
| **Access Switches** | **2960-24TT** (o PoE) | 2 | Switch di accesso per uffici e stabilimento industriale |
| **IP Phones** | **7960** | 3 | Terminali VoIP (Direzione, Supporto Tecnico, Produzione) |
| **End Devices** | **PC-PT** | 2 | Postazioni di lavoro client per il personale |
| **Smart Devices** | **Smart Device / TV** | 1 | Display digitale di linea per monitoraggio KPI e OEE |
| **Videosorveglianza**| **IP Camera** | 2 | Telecamere HD PoE per sicurezza e monitoraggio stabilimento |
| **Servers** | **Server-PT** | 3 | Active Directory (`.10`), NVR Storage (`.50`), Web DMZ (`.90.10`) |
| **Wireless** | **AccessPoint-PT** / WLC | 2 | Copertura Wi-Fi aziendale e ospiti |

---

## 2. Tabella di Cablaggio Completa (Cabling Guide)

> **Cavo utilizzato:** Cavo in rame diretto **Copper Straight-Through** (linea continua nera).

| Da Dispositivo | Porta | A Dispositivo | Porta | Note Architetturali |
| :--- | :--- | :--- | :--- | :--- |
| **ASA-FIREWALL-01** | `GigabitEthernet1/1` | **ISP Cloud** | `FastEthernet0` | Collegamento WAN / Internet |
| **ASA-FIREWALL-01** | `GigabitEthernet1/2` | **CORE-DIST-SW01** | `GigabitEthernet0/1` | Link Punto-Punto Routed (`10.0.0.1` ↔ `10.0.0.2`) |
| **ASA-FIREWALL-01** | `GigabitEthernet1/3` | **SRV-DMZ-WEB** | `FastEthernet0` | Zona DMZ pubblica isolata (`172.16.90.10`) |
| **CORE-DIST-SW01** | `GigabitEthernet0/2` | **SW-ACCESS-OFFICE** | `GigabitEthernet0/1` | Dorsale Trunk 802.1Q |
| **CORE-DIST-SW01** | `GigabitEthernet0/3` | **SW-ACCESS-FACTORY**| `GigabitEthernet0/1` | Dorsale Trunk 802.1Q |
| **CORE-DIST-SW01** | `FastEthernet0/10` | **SRV-DC-AD** | `FastEthernet0` | Server Active Directory / DNS (`192.168.50.10`) |
| **CORE-DIST-SW01** | `FastEthernet0/20` | **SRV-NVR-CCTV** | `FastEthernet0` | Server NVR Archiviazione Video (`192.168.50.50`) |
| **SW-ACCESS-OFFICE** | `FastEthernet0/1` | **IP Phone 1 (101)** | `Switch` | Porta verso il telefono IP |
| **IP Phone 1 (101)** | `PC` | **PC-Admin** | `FastEthernet0` | **Daisy-Chain:** Un solo cavo a muro per PC e Telefono! |
| **SW-ACCESS-OFFICE** | `FastEthernet0/2` | **IP Phone 2 (102)** | `Switch` | Porta verso il telefono supporto |
| **IP Phone 2 (102)** | `PC` | **PC-Support** | `FastEthernet0` | Connessione dati postazione 2 |
| **SW-ACCESS-OFFICE** | `FastEthernet0/11` | **Office-Printer-01** | `FastEthernet0` | Stampante laser di rete (VLAN 25) |
| **SW-ACCESS-FACTORY**| `FastEthernet0/1` | **IP Phone 3 (103)** | `Switch` | Telefono IP dello stabilimento |
| **SW-ACCESS-FACTORY**| `FastEthernet0/5` | **Smart TV Fabbrica** | `FastEthernet0` | Monitor cruscotto KPI linea (VLAN 60) |
| **SW-ACCESS-FACTORY**| `FastEthernet0/9` | **CAM-01-Office** | `FastEthernet0` | Telecamera CCTV PoE (VLAN 70) |
| **SW-ACCESS-FACTORY**| `FastEthernet0/10` | **CAM-02-Plant** | `FastEthernet0` | Telecamera CCTV PoE (VLAN 70) |
| **SW-ACCESS-FACTORY**| `FastEthernet0/17` | **Tornello RFID** | `FastEthernet0` | Controllo accessi industriale (VLAN 80) |

---

## 3. Passaggio Fondamentale: Alimentazione dei Telefoni IP e Telecamere

In Cisco Packet Tracer, i telefoni IP 7960 vengono inseriti di default senza alimentatore connesso:

1. Fai clic sul dispositivo **IP Phone 1**.
2. Si aprirà la scheda **Physical**.
3. In basso a destra noterai un connettore nero di alimentazione (**Power Adapter**).
4. Trascina con il mouse l'adattatore (Drag & Drop) e inseriscilo nella presa circolare posta sul retro del telefono.
5. Il LED si illuminerà di verde e sullo schermo LCD comparirà: `Configuring IP...`.
6. Ripeti la stessa operazione per **IP Phone 2** e **IP Phone 3**.

---

## 4. Applicazione dei File di Configurazione (CLI Deployment)

I file di configurazione pronti all'uso si trovano nella cartella `configs/`:

1. **Firewall ASA:** Apri la CLI di `ASA-FIREWALL-01` e incolla il contenuto di [`configs/firewall_asa.cfg`](../../configs/firewall_asa.cfg).
2. **Switch Core L3:** Apri la CLI di `CORE-DIST-SW01` e incolla il contenuto di [`configs/switch_core_multilayer.ios`](../../configs/switch_core_multilayer.ios).
3. **Switch Uffici:** Apri la CLI di `SW-ACCESS-OFFICE` e incolla [`configs/switch_access_office.ios`](../../configs/switch_access_office.ios).
4. **Switch Fabbrica:** Apri la CLI di `SW-ACCESS-FACTORY` e incolla [`configs/switch_access_factory.ios`](../../configs/switch_access_factory.ios).
5. **Router CME:** Apri la CLI di `R1-EDGE-CME` e incolla [`configs/router_cme.ios`](../../configs/router_cme.ios).

---

## 5. Configurazione delle Postazioni Client (End Devices)

### 1) PC Ufficio (PC-Admin & PC-Support):
- Fai clic su **PC-Admin** ➡️ Scheda **Desktop** ➡️ **IP Configuration**.
- Seleziona **DHCP**.
- Il PC riceverà l'IP `192.168.20.11` con gateway `192.168.20.1` e DNS `192.168.50.10`.

### 2) Smart TV di Linea:
- Fai clic su **Smart TV** ➡️ Scheda **Config** ➡️ Interfaccia **FastEthernet0**.
- Seleziona **DHCP**. Riceverà l'indirizzo `192.168.60.11`.

### 3) Server Farm:
- Fai clic su ciascun server e assegna gli indirizzi statici:
  - **SRV-DC-AD:** IP `192.168.50.10`, Mask `255.255.255.0`, Gateway `192.168.50.1`
  - **SRV-NVR-CCTV:** IP `192.168.50.50`, Mask `255.255.255.0`, Gateway `192.168.50.1`
  - **SRV-DMZ-WEB:** IP `172.16.90.10`, Mask `255.255.255.0`, Gateway `172.16.90.1`

---

## 6. Procedura di Verifica e Test di Funzionamento

### A) Verifica Chiamata Telefonica VoIP Reale:
1. Circa 60 secondi dopo la configurazione, sui telefoni appariranno gli interni registrati:
   - **IP Phone 1:** visualizza `101` in alto a destra.
   - **IP Phone 2:** visualizza `102`.
   - **IP Phone 3:** visualizza `103`.
2. Apri la scheda **GUI** su **IP Phone 1** e contemporaneamente su **IP Phone 2**.
3. Sul telefono **101**: fai clic sul ricevitore per sollevarlo (sentirai il segnale di linea libera *Dial Tone*).
4. Tramite il tastierino numerico componi l'interno **102**.
5. **Esito:** Il telefono **102** inizierà a squillare con spia rossa intermittente e visualizzerà a display `Ring In From 101`!
6. Solleva il ricevitore del telefono 102 per rispondere: lo stato cambierà all'istante in **Connected**!

### B) Test di Connettività e Navigazione (Ping & Web Test):
1. Da **PC-Admin**, apri il **Command Prompt** e digita:
   ```bash
   ping 192.168.50.50
   ```
   Riceverai risposta immediata (`TTL=127, time=1ms`) a conferma del corretto routing Inter-VLAN.
2. Sempre da PC-Admin, digita:
   ```bash
   ping 172.16.90.10
   ```
   Il pacchetto attraverserà lo switch core e l'ASA Firewall verso la DMZ con esito 100% positivo!
3. Apri il **Web Browser** su PC-Admin e inserisci l'URL `http://172.16.90.10`: si caricherà la pagina Web del portale aziendale.
