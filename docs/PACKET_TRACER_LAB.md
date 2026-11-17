# Step-by-Step Cisco Packet Tracer Implementation Guide

This guide walks through assembling and deploying the entire network infrastructure inside **Cisco Packet Tracer**.

---

## 1. Device Selection Inventory

Locate and drag the following network hardware from the bottom device tray into the workspace:

| Category | Model Name | Quantity | Target Label |
| :--- | :--- | :---: | :--- |
| **Routers** | `2811` or `2911` | 1 | `R1-EDGE-CME` |
| **Switches** | `2960-24TT` | 1 | `SW-ACCESS-01` |
| **IP Phones** | `7960` | 3 | `Phone-101`, `Phone-102`, `Phone-103` |
| **Computers** | `PC-PT` | 2 | `PC-Admin`, `PC-Engineering` |
| **Smart Devices** | `Smart Device` or `TV` | 1 | `Smart-TV-Factory` |
| **Servers** | `Server-PT` | 1 | `DataCenter-SRV` |
| **Wireless** | `AccessPoint-PT` | 1 | `Office-AP` |

---

## 2. Structured Cabling Matrix

Connect all devices using **Copper Straight-Through Cables** (solid black cable in the connections panel):

```
[R1-EDGE-CME] Gig0/0 <----------------------------> Gig0/1 [SW-ACCESS-01]
[SW-ACCESS-01] Fa0/1  <----------------------------> Switch [Phone-101]
[Phone-101]    PC     <----------------------------> FastEthernet0 [PC-Admin]
[SW-ACCESS-01] Fa0/2  <----------------------------> Switch [Phone-102]
[Phone-102]    PC     <----------------------------> FastEthernet0 [PC-Engineering]
[SW-ACCESS-01] Fa0/3  <----------------------------> Switch [Phone-103]
[SW-ACCESS-01] Fa0/10 <----------------------------> FastEthernet0 [Smart-TV-Factory]
[SW-ACCESS-01] Fa0/15 <----------------------------> FastEthernet0 [DataCenter-SRV]
[SW-ACCESS-01] Fa0/20 <----------------------------> Port0 [Office-AP]
```

---

## 3. Powering On Cisco 7960 IP Phones

Cisco IP Phones in Packet Tracer require manual DC power supply attachment:

1. Click on **Phone-101** to open its device inspection window.
2. Select the **Physical** tab.
3. Locate the black power adapter plug at the bottom-right corner of the window.
4. Drag and drop the power cord into the round DC power port on the lower-left rear of the phone.
5. The front panel display will power on and illuminate.
6. Repeat this procedure for **Phone-102** and **Phone-103**.

---

## 4. Applying Configurations

1. **Router Configuration:**
   - Open **R1-EDGE-CME** -> Click the **CLI** tab.
   - If prompted with `Continue with configuration dialog? [yes/no]:`, type `no` and press Enter.
   - Paste the entire contents of [`configs/router_cme.ios`](../configs/router_cme.ios).

2. **Switch Configuration:**
   - Open **SW-ACCESS-01** -> Click the **CLI** tab.
   - Press Enter to enter CLI mode.
   - Paste the entire contents of [`configs/switch_access.ios`](../configs/switch_access.ios).

---

## 5. End Device Verification

1. **Workstations (PC-Admin & PC-Engineering):**
   - Click each PC -> Navigate to **Desktop** -> **IP Configuration**.
   - Select **DHCP**. Verify that an IP address in the `192.168.20.0/24` range is leased successfully.
2. **Smart TV / Production Display:**
   - Click the Smart TV -> Navigate to **Config** -> **FastEthernet0**.
   - Select **DHCP**. Verify an IP is assigned within the `192.168.60.0/24` subnet.
3. **Server:**
   - Configure a static IP:
     - **IP Address:** `192.168.50.10`
     - **Subnet Mask:** `255.255.255.0`
     - **Default Gateway:** `192.168.50.1`

---

## 6. Live VoIP Telephony Testing

1. Once the IP Phones finish their registration sequence, inspect their LCD screens:
   - Phone 1 displays its assigned directory number: **101** in the top-right corner.
   - Phone 2 displays: **102**.
   - Phone 3 displays: **103**.
2. **Placing a Call:**
   - Open the **GUI** tab for **Phone-101** and **Phone-102** side-by-side.
   - Click the handset of **Phone-101** to take it off-hook (dial tone sounds).
   - Dial `102` using the keypad.
   - Notice **Phone-102** begins ringing immediately with a flashing red LED and displays `Ring In From 101`.
   - Click the handset of **Phone-102** to answer the call. The screen status will transition to `Connected`.
