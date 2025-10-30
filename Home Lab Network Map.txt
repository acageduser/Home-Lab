Home Lab Network Map.txt
Documentation for the physical layout and rules of my home network.

View-only Diagram Link:
https://www.mermaidchart.com/d/078828cc-05ef-48f1-b4c9-1633ba9b3beb

Rendered Diagram (Mermaid)
- Keep this repo public-safe: do not include passwords, MACs, serials, public IP/hostname, or edit URLs.

```mermaid
---
config:
  theme: dark
  layout: elk
---
flowchart TB
 subgraph LEGEND["Key"]
        L10["VLAN 10 – Trusted PCs<br>192.168.20.0/24"]
        L30["VLAN 30 – Media/IoT<br>192.168.30.0/24"]
        L40["VLAN 40 – Guest<br>192.168.40.0/24"]
        LINF["Infra / WAN / Trunk"]
 end

 subgraph CORE["Dining Room"]
        R["AX1800 Router<br>192.168.20.1"]
        S["Server PC i5-8600<br>192.168.20.10"]
 end

 subgraph REMOTE["Bedroom"]
        SW["TP-Link TL-SG108E (8-port)<br>Mgmt: 192.168.20.3"]
        P1["PC 1<br>192.168.20.20"]
        L1["Laptop<br>192.168.20.21"]
        P2["PC 2<br>192.168.20.22"]
        TV["TV<br>192.168.30.30"]
        NS["Nintendo Switch 2<br>192.168.30.31"]
 end

 %% Wi-Fi clients (examples)
 G1["Roommate Phone<br>DHCP on join"]:::v40
 G2["Guest Phone<br>DHCP on join"]:::v40

 %% WAN
 ISP((("Internet"))) -- |30 Mb/s| --> M["Modem/ONT"]
 M -- |WAN| --> R

 %% Wired links (preferred label style)
 R -- Cat6 | 1 ft | 1G --> S
 R -- LAN1 → SW:1 | Cat6 | 100 ft --> SW
 SW -- Port 2 | Cat6 | 6 ft --> P1
 SW -- Port 3 | Cat6 | 6 ft --> L1
 SW -- Port 4 | Cat6 | 20 ft --> P2
 SW -- Port 5 | Cat6 | 25 ft --> TV
 SW -- Port 6 | Cat6 | 25 ft --> NS

 %% Wi-Fi associations
 R -- Home SSID --> L1
 R -- Guest SSID --> G1
 R -- Guest SSID --> G2

 %% Classes
 L10:::v10
 L30:::v30
 L40:::v40
 LINF:::infra
 R:::infra
 S:::v10
 SW:::infra
 P1:::v10
 L1:::v10
 P2:::v10
 TV:::v30
 NS:::v30
 G1:::v40
 G2:::v40

 %% Styles
 classDef v10 fill:#cfe9ff,stroke:#1d70b8,color:#0b0b0b
 classDef v30 fill:#ffe6cc,stroke:#cc7000,color:#0b0b0b
 classDef v40 fill:#e6ffe6,stroke:#2e8b57,color:#0b0b0b
 classDef infra fill:#e9e9e9,stroke:#555,color:#0b0b0b
```

SETUP GOALS
- Keep my gear safe.
- Simple guest use.
- One source of truth.

ADDRESSING
- VLAN 10 – Trusted PCs → 192.168.20.0/24
  - Router: 192.168.20.1
  - Switch mgmt: 192.168.20.3
  - Server: 192.168.20.10
  - PC1: 192.168.20.20
  - Laptop: 192.168.20.21
  - PC2: 192.168.20.22
  - DHCP pool: 192.168.20.100–199
- VLAN 30 – Media/IoT → 192.168.30.0/24
  - TV: 192.168.30.30
  - Switch 2: 192.168.30.31
  - DHCP pool: 192.168.30.100–199
- VLAN 40 – Guest → 192.168.40.0/24
  - Phones join by Wi-Fi only
  - DHCP pool: 192.168.40.100–199

SWITCH PORT MAP (TL-SG108E)
- [Port 1 | Trunk | Tagged: 10,30,40 | PVID 10] ← uplink to Router LAN1
- [Port 2 | Access | VLAN 10] → PC 1
- [Port 3 | Access | VLAN 10] → Laptop
- [Port 4 | Access | VLAN 10] → PC 2
- [Port 5 | Access | VLAN 30] → TV
- [Port 6 | Access | VLAN 30] → Nintendo Switch 2
- [Port 7 | Spare]
- [Port 8 | Spare]
- Management VLAN = 10. Switch IP = 192.168.20.3.

WI-FI SSIDs
- “Home” SSID → VLAN 10
- “Guest” SSID → VLAN 40 with “guest isolation / block LAN access” on

DHCP RESERVATIONS (on router)
- Reserve all fixed IPs by MAC.
- Lease time: 7 days.
- Keep dynamic clients in 100–199 on each VLAN.

ROUTER FIREWALL (default-deny to VLAN 10)
- For VLAN 40 and VLAN 30
  1) Allow DHCP/DNS/NTP to router (UDP 67–68, TCP/UDP 53, UDP 123)
  2) Block 192.168.20.0/24 and block other RFC1918 ranges except own /24
  3) Allow any to WAN
- For VLAN 10
  - Allow WAN
  - Allow router UI and switch mgmt (192.168.20.1, 192.168.20.3)
  - Block to VLAN 30 and VLAN 40

SERVER LOCKDOWN
- Windows (SMB):
  - Private profile only; “File and Printer Sharing (SMB-In)” → Scope = 192.168.20.0/24
  - Disable SMBv1; use accounts, not “Everyone”
- Linux (Samba):
  interfaces = 192.168.20.10/24
  bind interfaces only = yes
  hosts allow = 192.168.20.0/24
  hosts deny = 0.0.0.0/0
  server min protocol = SMB2

TEST PLAN
- Link LEDs show 1G on wired ports
- Guests cannot open http://192.168.20.10 or \\192.168.20.10\share
- PC1 can open server shares
- iperf3 LAN ≈ 940 Mb/s; internet ≈ 30 Mb/s (ISP cap)

CABLE + PORT LABELS
- [LAN1 → SW:1 | Cat6 | 100 ft | 1G]
- [Port 2 | Cat6 | 6 ft | 1G] PC 1
- [Port 3 | Cat6 | 6 ft | 1G] Laptop
- [Port 4 | Cat6 | 20 ft | 1G] PC 2
- [Port 5 | Cat6 | 25 ft | 1G] TV
- [Port 6 | Cat6 | 25 ft | 1G] Switch 2

SECURITY BASELINE
- Strong admin creds • Disable UPnP • Turn off WPS
- Back up router and switch configs
- Firmware updates twice a year
- Management only on VLAN 10

CHANGE LOG
- v0.1 baseline map and policy
- v0.2 add Guest SSID and firewall blocks
- v0.3 add casting exception (if needed)

%%eof
