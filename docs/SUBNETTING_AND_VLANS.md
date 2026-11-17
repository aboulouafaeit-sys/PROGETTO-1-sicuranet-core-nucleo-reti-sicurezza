# Complete Enterprise & Industrial IP Addressing Plan

## 1. Master VLAN Matrix

| VLAN | Segment Identifier | Subnet Prefix | Subnet Mask | Default Gateway | Usable Host Range | Target Devices & Systems |
| :---: | :--- | :--- | :--- | :--- | :--- | :--- |
| **10** | `MGMT_INFRA` | `192.168.10.0/24` | `255.255.255.0` | `192.168.10.1` | `192.168.10.2 - 254` | Switches, Routers, Firewalls, Out-of-band management |
| **20** | `DATA_STAFF` | `192.168.20.0/24` | `255.255.255.0` | `192.168.20.1` | `192.168.20.11 - 254` | Administrative, Finance, and Engineering Workstations |
| **25** | `PRINTERS` | `192.168.25.0/24` | `255.255.255.0` | `192.168.25.1` | `192.168.25.11 - 254` | Departmental Network Printers & Multi-Function Scanners |
| **30** | `VOICE_VOIP` | `192.168.30.0/24` | `255.255.255.0` | `192.168.30.1` | `192.168.30.11 - 254` | Cisco 7960 IP Phones (802.1p CoS 5 Voice Priority) |
| **40** | `CORP_WIFI` | `192.168.40.0/24` | `255.255.255.0` | `192.168.40.1` | `192.168.40.11 - 254` | Corporate Laptops & Executive Mobile Devices |
| **50** | `SERVERS_DC` | `192.168.50.0/24` | `255.255.255.0` | `192.168.50.1` | `192.168.50.2 - 254` | AD/DNS (`.10`), ERP Database (`.20`), NVR Server (`.50`) |
| **60** | `SMART_TVS` | `192.168.60.0/24` | `255.255.255.0` | `192.168.60.1` | `192.168.60.11 - 254` | Factory Production Overhead Dashboards & Smart TVs |
| **70** | `CCTV_CAMERAS`| `192.168.70.0/24`| `255.255.255.0` | `192.168.70.1` | `192.168.70.11 - 254` | PoE Surveillance IP Cameras (Factory & Office) |
| **80** | `FACTORY_OT` | `192.168.80.0/24` | `255.255.255.0` | `192.168.80.1` | `192.168.80.11 - 254` | Industrial PLCs, Environmental Sensors, Door Access |
| **90** | `DMZ_PUBLIC` | `172.16.90.0/24` | `255.255.255.0` | `172.16.90.1` | `172.16.90.2 - 254` | Public Customer Portal & External Web Servers |
| **99** | `GUEST_WIFI` | `192.168.99.0/24` | `255.255.255.0` | `192.168.99.1` | `192.168.99.11 - 254` | Isolated Visitor Wi-Fi (Direct Internet Only) |

---

## 2. Dynamic Host Configuration (DHCP) Scopes Summary

- **Staff Data:** `192.168.20.0/24` | Gateway: `192.168.20.1` | DNS: `192.168.50.10`
- **VoIP Voice:** `192.168.30.0/24` | Gateway: `192.168.30.1` | Option 150: `192.168.30.1`
- **Smart TVs:** `192.168.60.0/24` | Gateway: `192.168.60.1` | DNS: `8.8.8.8`
- **CCTV Cameras:** `192.168.70.0/24` | Gateway: `192.168.70.1` | NVR Target: `192.168.50.50`
- **Industrial OT:** `192.168.80.0/24` | Gateway: `192.168.80.1`
- **Guest Wi-Fi:** `192.168.99.0/24` | Gateway: `192.168.99.1` | DNS: `8.8.8.8`
