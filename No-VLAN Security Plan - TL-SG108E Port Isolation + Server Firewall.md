# No-VLAN Security Plan - TL-SG108E Port Isolation + Server Firewall
I’m keeping a flat LAN (no VLAN router). I still want isolation. I do two things:
1) **Port isolation** on the TL-SG108E in the bedroom.
2) **Server firewall** that only allows my PCs.

---

## A) TL-SG108E port isolation (a.k.a. MTU VLAN / Protected Ports)
**Goal:** Ports **2–6** cannot talk to each other. All can reach **Port 1** (uplink → router).  
**Result:** Bedroom devices don’t see each other. Everyone still has internet.

There are two UI variants on Easy Smart. One of these matches my firmware.

### Option 1 — “Port Isolation” page
1. Log in to the switch web UI.  
2. Go to **Port** → **Port Isolation**.  
3. For **Port 1** (uplink): allow forwarding to **2,3,4,5,6**.  
4. For **Ports 2–6**: allow forwarding **only to Port 1**.  
5. Apply. Save config.

### Option 2 — “VLAN” → “MTU VLAN” page
1. Enable **MTU VLAN**.  
2. Set **Uplink Port = 1**.  
3. Add **Ports 2–6** as MTU member/isolated ports.  
4. Apply. Save config.

**Port map I use**
- Port 1: uplink to router LAN1  
- Port 2: PC 1  
- Port 3: Laptop  
- Port 4: PC 2  
- Port 5: TV  
- Port 6: Nintendo Switch 2  
- Ports 7–8: spare

**Quick tests**
- `ping` between P1 and L1 should fail.  
- Both can still browse the internet.

---

## B) Server lock-down
I allow file access only from my trusted PCs.

### Windows (SMB)
1. Open **Windows Defender Firewall with Advanced Security**.  
2. In **Inbound Rules**, find **File and Printer Sharing (SMB-In)**.  
3. Right-click → **Properties** → **Scope** tab.  
4. **Remote IP addresses** → **These IP addresses** → add:  
   - `192.168.20.20` (PC1)  
   - `192.168.20.21` (Laptop)  
   - `192.168.20.22` (PC2)  
5. Ensure **Profile = Private** only.  
6. Disable **SMBv1**. Use accounts, not “Everyone”.

### Linux + Samba
Add to `smb.conf`:
```ini
interfaces = 192.168.20.10/24
bind interfaces only = yes
hosts allow = 192.168.20.20, 192.168.20.21, 192.168.20.22
hosts deny = 0.0.0.0/0
server min protocol = SMB2
```
Restart Samba.

---

## C) Wi-Fi
- One SSID. One password. No guest SSID.  
- My laptop can still reach the server because it’s allowed by IP.  
- Roommate phones just use the internet. Server blocks them.

---

## D) Ops and safety
- Export and save the switch config after changes.  
- Backup the server. Test a restore monthly.  
- No UPnP. Strong admin passwords.  
- If I add a VLAN router later, I can switch to the VLAN design without moving cables.

---

## E) Verification checklist
- [ ] Bedroom: devices can’t ping each other.  
- [ ] Internet works on all devices.  
- [ ] From PC1/Laptop/PC2: `\\192.168.20.10\share` works.  
- [ ] From any phone/TV: server SMB and HTTP do not open.  
- [ ] Link LEDs = 1G on all wired ports.
