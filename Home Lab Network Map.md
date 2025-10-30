# Home Lab Network Map
Flat LAN. Archer AX21 stays stock (no 802.1Q). Bedroom switch uses **port isolation**. My server allows SMB only from my PCs.

View-only Diagram: <https://www.mermaidchart.com/d/078828cc-05ef-48f1-b4c9-1633ba9b3beb>

```mermaid
---
config:
  theme: dark
  layout: elk
---
flowchart TB
  %% Core room
  subgraph CORE["Dining Room"]
    R["AX1800 Router (AX21)<br/>192.168.20.1"]
    S["Server PC i5-8600<br/>192.168.20.10"]
  end

  %% Remote room
  subgraph REMOTE["Bedroom"]
    SW["TP-Link TL-SG108E (8-port)<br/>Mgmt: 192.168.20.3<br/><i>Port Isolation: Ports 2–6 isolated; Port 1 = uplink</i>"]
    P1["PC 1<br/>192.168.20.20"]
    L1["Laptop<br/>192.168.20.21"]
    P2["PC 2<br/>192.168.20.22"]
    TV["TV<br/>192.168.20.30"]
    NS["Nintendo Switch 2<br/>192.168.20.31"]
  end

  %% WAN
  ISP((("Internet"))) -- |30 Mb/s| --> M["Modem/ONT"]
  M -- |WAN| --> R

  %% Wired links (my label style)
  R -- Cat6 | 1 ft | 1G --> S
  R -- LAN1 → SW:1 | Cat6 | 100 ft | 1G --> SW
  SW -- Port 2 | Cat6 | 6 ft | 1G --> P1
  SW -- Port 3 | Cat6 | 6 ft | 1G --> L1
  SW -- Port 4 | Cat6 | 20 ft | 1G --> P2
  SW -- Port 5 | Cat6 | 25 ft | 1G --> TV
  SW -- Port 6 | Cat6 | 25 ft | 1G --> NS
```
