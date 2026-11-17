# دليل أوامر الـ CLI وسجل التنفيذ الشامل (Cisco CLI Execution Guide)
### التوثيق الكامل للأوامر المنفذة على الراوترات، الفايروول، والسويتشات مع نتائج الفحص

هذا الدليل مخصص لشرح وتوثيق كل أمر كُتب ونُفّذ داخل شاشات الـ CLI في بيئة العمل وأجهزة Cisco، مع توضيح سبب كتابة كل أمر ومخرجات التحقق (`Show Commands`).

---

## 📑 فهرس المحتويات
1. [أوامر جدار الحماية (Cisco ASA 5506-X Firewall)](#1-أوامر-جدار-الحماية-cisco-asa-5506-x-firewall)
2. [أوامر سويتش التوجيه الرئيسي (Catalyst 3560/3650 Core L3 Switch)](#2-أوامر-سويتش-التوجيه-الرئيسي-catalyst-35603650-core-l3-switch)
3. [أوامر راوتر وسنترال المكالمات (Cisco 2811/2911 CME PBX)](#3-أوامر-راوتر-وسنترال-المكالمات-cisco-28112911-cme-pbx)
4. [أوامر سويتش المكاتب (Catalyst 2960 Office Switch)](#4-أوامر-سويتش-المكاتب-catalyst-2960-office-switch)
5. [أوامر سويتش المصنع الصناعي (Catalyst 2960 / IE Factory Switch)](#5-أوامر-سويتش-المصنع-الصناعي-catalyst-2960--ie-factory-switch)
6. [أوامر الفحص والتحقق الكامل (Verification & Troubleshooting)](#6-أوامر-الفحص-والتحقق-الكامل-verification--troubleshooting)

---

## 1. أوامر جدار الحماية (Cisco ASA 5506-X Firewall)

جدار الحماية هو خط الدفاع الأول لعزل الشبكة الخارجية (الإنترنت)، المنطقة المعزولة (DMZ)، والشبكة المؤسسية الداخلية (Inside).

### أ) تسلسل الأوامر المنفذة:
```cisco
ciscoasa> enable
ciscoasa# configure terminal
ciscoasa(config)# hostname ASA-FIREWALL-01

! 1. إعداد المنافذ وتحديد مستويات الأمان (Security Levels)
! المنفذ الخارجي المتصل بمزود الخدمة (مستوى أمان 0 = غير موثوق)
ASA-FIREWALL-01(config)# interface GigabitEthernet1/1
ASA-FIREWALL-01(config-if)# nameif outside
ASA-FIREWALL-01(config-if)# security-level 0
ASA-FIREWALL-01(config-if)# ip address dhcp setroute
ASA-FIREWALL-01(config-if)# no shutdown
ASA-FIREWALL-01(config-if)# exit

! المنفذ الداخلي المتصل بسويتش الكور (مستوى أمان 100 = موثوق بالكامل)
ASA-FIREWALL-01(config)# interface GigabitEthernet1/2
ASA-FIREWALL-01(config-if)# nameif inside
ASA-FIREWALL-01(config-if)# security-level 100
ASA-FIREWALL-01(config-if)# ip address 10.0.0.1 255.255.255.252
ASA-FIREWALL-01(config-if)# no shutdown
ASA-FIREWALL-01(config-if)# exit

! منفذ المنطقة منزوعة السلاح DMZ (مستوى أمان 50 = شبه موثوق)
ASA-FIREWALL-01(config)# interface GigabitEthernet1/3
ASA-FIREWALL-01(config-if)# nameif dmz
ASA-FIREWALL-01(config-if)# security-level 50
ASA-FIREWALL-01(config-if)# ip address 172.16.90.1 255.255.255.0
ASA-FIREWALL-01(config-if)# no shutdown
ASA-FIREWALL-01(config-if)# exit

! 2. إنشاء كائنات الشبكة (Network Objects)
ASA-FIREWALL-01(config)# object network OBJ_INTERNAL_CORP
ASA-FIREWALL-01(config-network-object)# subnet 192.168.0.0 255.255.0.0
ASA-FIREWALL-01(config-network-object)# exit

ASA-FIREWALL-01(config)# object network OBJ_PUBLIC_WEB_SERVER
ASA-FIREWALL-01(config-network-object)# host 172.16.90.10
ASA-FIREWALL-01(config-network-object)# exit

! 3. تفعيل ترجمة العناوين الديناميكية (Dynamic PAT) لخروج الموظفين للإنترنت
ASA-FIREWALL-01(config)# nat (inside,outside) source dynamic OBJ_INTERNAL_CORP interface

! 4. قائمة التحكم بالوصول (ACL) للسماح بتصفح موقع الشركة فقط في DMZ
ASA-FIREWALL-01(config)# access-list ACL_OUTSIDE_IN extended permit tcp any host 172.16.90.10 eq 80
ASA-FIREWALL-01(config)# access-list ACL_OUTSIDE_IN extended permit tcp any host 172.16.90.10 eq 443
ASA-FIREWALL-01(config)# access-list ACL_OUTSIDE_IN extended permit icmp any any echo-reply
ASA-FIREWALL-01(config)# access-group ACL_OUTSIDE_IN in interface outside

! 5. تفعيل الفحص العميق للحزم (Deep Packet Inspection)
ASA-FIREWALL-01(config)# policy-map global_policy
ASA-FIREWALL-01(config-pmap)# class inspection_default
ASA-FIREWALL-01(config-pmap-c)# inspect icmp
ASA-FIREWALL-01(config-pmap-c)# inspect http
ASA-FIREWALL-01(config-pmap-c)# inspect sip
ASA-FIREWALL-01(config-pmap-c)# inspect dns
ASA-FIREWALL-01(config-pmap-c)# exit
ASA-FIREWALL-01(config-pmap)# exit

! 6. المسار الثابت (Static Route) لتوجيه حركة الشبكة الداخلية إلى سويتش الكور
ASA-FIREWALL-01(config)# route inside 192.168.0.0 255.255.0.0 10.0.0.2
ASA-FIREWALL-01(config)# write memory
```

### ب) مخرجات التحقق من الفايروول:
```cisco
ASA-FIREWALL-01# show nameif
Interface                Name                     Security
GigabitEthernet1/1       outside                     0
GigabitEthernet1/2       inside                    100
GigabitEthernet1/3       dmz                        50

ASA-FIREWALL-01# show access-list
access-list ACL_OUTSIDE_IN line 1 extended permit tcp any host 172.16.90.10 eq www (hitcnt=248)
access-list ACL_OUTSIDE_IN line 2 extended permit tcp any host 172.16.90.10 eq 443 (hitcnt=1420)
access-list ACL_OUTSIDE_IN line 3 extended permit icmp any any echo-reply (hitcnt=16)

ASA-FIREWALL-01# show xlate
TCP PAT from inside:192.168.20.11/51234 to outside:203.0.113.2/51234 flags e idle 0:00:04
```

---

## 2. أوامر سويتش التوجيه الرئيسي (Catalyst 3560/3650 Core L3 Switch)

يقوم هذا السويتش بجميع عمليات التوجيه بين الشبكات الفرعية بسرعة المعالج (Wire-speed Inter-VLAN Routing) عبر واجهات الـ SVI.

### أ) تسلسل الأوامر المنفذة:
```cisco
Switch> enable
Switch# configure terminal
Switch(config)# hostname CORE-DIST-SW01

! 1. تفعيل محرك التوجيه في السويتش
CORE-DIST-SW01(config)# ip routing

! 2. إنشاء جميع الـ VLANs وتسميتها
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

! 3. بناء البوابات الافتراضية لكل شبكة (SVIs)
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

! 4. تحويل منفذ الفايروول لمنفذ موجه (Routed Port) وإعداد البوابة الافتراضية
CORE-DIST-SW01(config)# interface GigabitEthernet0/1
CORE-DIST-SW01(config-if)# no switchport
CORE-DIST-SW01(config-if)# ip address 10.0.0.2 255.255.255.252
CORE-DIST-SW01(config-if)# no shutdown
CORE-DIST-SW01(config-if)# exit

CORE-DIST-SW01(config)# ip route 0.0.0.0 0.0.0.0 10.0.0.1

! 5. تفعيل خطوط الـ Trunk لنقل جميع الـ VLANs لسويتشات الوصول
CORE-DIST-SW01(config)# interface range GigabitEthernet0/2 - 4
CORE-DIST-SW01(config-if-range)# switchport trunk encapsulation dot1q
CORE-DIST-SW01(config-if-range)# switchport mode trunk
CORE-DIST-SW01(config-if-range)# switchport trunk allowed vlan 10,20,25,30,40,50,60,70,80,99
CORE-DIST-SW01(config-if-range)# no shutdown
CORE-DIST-SW01(config-if-range)# exit
CORE-DIST-SW01(config)# do write memory
```

### ب) مخرجات التحقق من سويتش الكور:
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
```

---

## 3. أوامر راوتر وسنترال المكالمات (Cisco 2811/2911 CME PBX)

يقوم الراوتر بدور السنترال الداخلي، وتوزيع العناوين التلقائية (DHCP) مع تزويد هواتف الـ IP برقم خادم الـ TFTP عبر `Option 150`.

### أ) تسلسل الأوامر المنفذة:
```cisco
Router> enable
Router# configure terminal
Router(config)# hostname R1-EDGE-CME

! 1. بناء الواجهات الفرعية (Sub-Interfaces 802.1Q)
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

! 2. إنشاء نطاقات الـ DHCP
R1-EDGE-CME(config)# ip dhcp excluded-address 192.168.20.1 192.168.20.10
R1-EDGE-CME(config)# ip dhcp excluded-address 192.168.30.1 192.168.30.10
R1-EDGE-CME(config)# ip dhcp excluded-address 192.168.60.1 192.168.60.10

! نطاق حواسيب الموظفين
R1-EDGE-CME(config)# ip dhcp pool POOL-STAFF-DATA
R1-EDGE-CME(dhcp-config)# network 192.168.20.0 255.255.255.0
R1-EDGE-CME(dhcp-config)# default-router 192.168.20.1
R1-EDGE-CME(dhcp-config)# dns-server 192.168.50.10

! نطاق هواتف الـ IP (خيار Option 150 يوجه الهاتف لعنوان السنترال لتحميل السوفتوير)
R1-EDGE-CME(config)# ip dhcp pool POOL-VOIP-VOICE
R1-EDGE-CME(dhcp-config)# network 192.168.30.0 255.255.255.0
R1-EDGE-CME(dhcp-config)# default-router 192.168.30.1
R1-EDGE-CME(dhcp-config)# option 150 ip 192.168.30.1

! نطاق شاشات العرض الذكية
R1-EDGE-CME(config)# ip dhcp pool POOL-SMART-TVS
R1-EDGE-CME(dhcp-config)# network 192.168.60.0 255.255.255.0
R1-EDGE-CME(dhcp-config)# default-router 192.168.60.1
R1-EDGE-CME(dhcp-config)# exit

! 3. تشغيل خدمة الهاتف وسنترال سيسكو (Telephony Service)
R1-EDGE-CME(config)# telephony-service
R1-EDGE-CME(config-telephony)# max-ephones 10
R1-EDGE-CME(config-telephony)# max-dn 10
R1-EDGE-CME(config-telephony)# ip source-address 192.168.30.1 port 2000
R1-EDGE-CME(config-telephony)# auto assign 1 to 10
R1-EDGE-CME(config-telephony)# create cnf-files
R1-EDGE-CME(config-telephony)# exit

! 4. تعريف الأرقام الداخلية للمكاتب
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

### ب) مخرجات التحقق من الراوتر:
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

## 4. أوامر سويتش المكاتب (Catalyst 2960 Office Switch)

```cisco
Switch> enable
Switch# configure terminal
Switch(config)# hostname SW-ACCESS-OFFICE

SW-ACCESS-OFFICE(config)# vlan 10,20,25,30,40
SW-ACCESS-OFFICE(config)# exit

! ربط الحاسوب والهاتف بكابل واحد على نفس المنفذ
SW-ACCESS-OFFICE(config)# interface range FastEthernet0/1 - 10
SW-ACCESS-OFFICE(config-if-range)# switchport mode access
SW-ACCESS-OFFICE(config-if-range)# switchport access vlan 20
SW-ACCESS-OFFICE(config-if-range)# switchport voice vlan 30
SW-ACCESS-OFFICE(config-if-range)# spanning-tree portfast
SW-ACCESS-OFFICE(config-if-range)# mls qos trust cos
SW-ACCESS-OFFICE(config-if-range)# no shutdown
SW-ACCESS-OFFICE(config-if-range)# exit

! منافذ الطابعات الشبكية
SW-ACCESS-OFFICE(config)# interface range FastEthernet0/11 - 14
SW-ACCESS-OFFICE(config-if-range)# switchport mode access
SW-ACCESS-OFFICE(config-if-range)# switchport access vlan 25
SW-ACCESS-OFFICE(config-if-range)# spanning-tree portfast
SW-ACCESS-OFFICE(config-if-range)# no shutdown
SW-ACCESS-OFFICE(config-if-range)# exit

! كابل الـ Trunk الصاعد إلى سويتش الكور
SW-ACCESS-OFFICE(config)# interface GigabitEthernet0/1
SW-ACCESS-OFFICE(config-if)# switchport mode trunk
SW-ACCESS-OFFICE(config-if)# switchport trunk allowed vlan 10,20,25,30,40,99
SW-ACCESS-OFFICE(config-if)# no shutdown
SW-ACCESS-OFFICE(config-if)# exit
SW-ACCESS-OFFICE(config)# do write memory
```

---

## 5. أوامر سويتش المصنع الصناعي (Catalyst 2960 / IE Factory Switch)

```cisco
Switch> enable
Switch# configure terminal
Switch(config)# hostname SW-ACCESS-FACTORY

SW-ACCESS-FACTORY(config)# vlan 10,30,60,70,80
SW-ACCESS-FACTORY(config)# exit

! شاشات العرض الذكية لمؤشرات الإنتاج
SW-ACCESS-FACTORY(config)# interface range FastEthernet0/5 - 8
SW-ACCESS-FACTORY(config-if-range)# switchport mode access
SW-ACCESS-FACTORY(config-if-range)# switchport access vlan 60
SW-ACCESS-FACTORY(config-if-range)# spanning-tree portfast
SW-ACCESS-FACTORY(config-if-range)# no shutdown
SW-ACCESS-FACTORY(config-if-range)# exit

! كاميرات المراقبة الأمنية عالية الدقة بتقنية PoE
SW-ACCESS-FACTORY(config)# interface range FastEthernet0/9 - 16
SW-ACCESS-FACTORY(config-if-range)# switchport mode access
SW-ACCESS-FACTORY(config-if-range)# switchport access vlan 70
SW-ACCESS-FACTORY(config-if-range)# spanning-tree portfast
SW-ACCESS-FACTORY(config-if-range)# no shutdown
SW-ACCESS-FACTORY(config-if-range)# exit

! حساسات المصنع وأبواب الدخول الذكية RFID
SW-ACCESS-FACTORY(config)# interface range FastEthernet0/17 - 24
SW-ACCESS-FACTORY(config-if-range)# switchport mode access
SW-ACCESS-FACTORY(config-if-range)# switchport access vlan 80
SW-ACCESS-FACTORY(config-if-range)# spanning-tree portfast
SW-ACCESS-FACTORY(config-if-range)# no shutdown
SW-ACCESS-FACTORY(config-if-range)# exit

! كابل الـ Trunk الصاعد لسويتش الكور
SW-ACCESS-FACTORY(config)# interface GigabitEthernet0/1
SW-ACCESS-FACTORY(config-if)# switchport mode trunk
SW-ACCESS-FACTORY(config-if)# switchport trunk allowed vlan 10,30,60,70,80
SW-ACCESS-FACTORY(config-if)# no shutdown
SW-ACCESS-FACTORY(config-if)# exit
SW-ACCESS-FACTORY(config)# do write memory
```

---

## 6. أوامر الفحص والتحقق الكامل (Verification & Troubleshooting)

| الجهاز | أمر الفحص | الوظيفة والنتيجة المتوقعة |
| :--- | :--- | :--- |
| **Cisco ASA** | `show nameif` | التأكد من أسماء المناطق ومستويات الأمان (outside=0, inside=100, dmz=50) |
| **Cisco ASA** | `show access-list` | التحقق من عدد مرات مطابقة قواعد الحماية (`hitcnt`) |
| **Cisco ASA** | `show xlate` | عرض جلسات الـ NAT/PAT المترجمة لخروج الأجهزة للإنترنت |
| **Core Switch** | `show ip route` | التأكد من وجود مسار افتراضي `0.0.0.0/0` وشبكات الـ Connected الـ 10 |
| **Core Switch** | `show interfaces trunk` | التأكد من تمرير الـ VLANs عبر منافذ الـ Uplink |
| **CME Router** | `show telephony-service ephone` | فحص تسجيل هواتف الـ IP وحالتها (`CONNECTED` أو `IDLE`) |
| **CME Router** | `show ip dhcp binding` | عرض الأجهزة التي أخذت IPs تلقائية من السيرفر |
| **Workstation** | `ping 192.168.50.50` | فحص الوصول لسيرفر كاميرات المراقبة (NVR) عبر الـ Inter-VLAN |
| **Workstation** | `ping 172.16.90.10` | فحص الوصول لسيرفر الـ Web في منطقة DMZ عبر الفايروول |
