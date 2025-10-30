# Home Networking To‑Do List
**Author:** Ryan  
**Lab PC:** HP ProDesk 600 G4 Mini (i5‑8500T) — <https://www.ebay.com/itm/226990879842>

---

## 0) GROUND RULES
**Minimum:** Mini PC  
**Recommended:** Mini PC + UPS

- [ ] Keep notes in Git (configs, `.env.example`, `README.md`).
- [ ] One change at a time → test → commit → move on.
- [ ] Use Tailscale for remote access. No router port‑forwards.
- [ ] UPS on the server. Power‑on‑after‑power‑loss in BIOS. Enable WoL.

---

## 1) BASE OS + REMOTE CONTROL
**Minimum:** Mini PC (headless), Ethernet  
**Recommended:** Mini PC + UPS + spare HDMI/keyboard for first boot  
**Pick:** Ubuntu Server LTS (Docker host)

- [ ] Set static DHCP reservation on router.
- [ ] Install SSH:
  ```bash
  sudo apt update && sudo apt install -y openssh-server
  ```
- [ ] Install Tailscale and enable SSH:
  ```bash
  sudo apt install -y tailscale
  sudo tailscale up --ssh
  ```
- [ ] Enable WoL in BIOS; then in OS:
  ```bash
  sudo ethtool -s <nic> wol g
  ```
- [ ] Create an admin user; add to sudo:
  ```bash
  sudo adduser <user>
  sudo usermod -aG sudo <user>
  ```

**Remote control from gaming PC**
- [ ] Use SSH for shell and web UIs for apps.
- [ ] If desktop needed: RustDesk or RDP to a Windows VM (optional later).  
  Tutorial: <https://www.youtube.com/watch?v=x9A7MuAvDlQ>

---

## 2) CONTAINER RUNTIME
**Minimum:** Mini PC  
**Recommended:** Mini PC + Portainer for GUI

- [ ] Install Docker + Compose plugin:
  ```bash
  sudo apt install -y docker.io docker-compose-plugin
  ```
- [ ] Add user to docker group and re‑log:
  ```bash
  sudo usermod -aG docker $USER
  ```
- [ ] Optional GUI: Portainer.

---

## 3) BACKUPS FIRST (LOCAL ONLY)
**Minimum:** Mini PC + one USB SSD  
**Recommended:** Two USB SSDs (rotate) + weekly test restore

- [ ] Tool: restic (or borg).
- [ ] Target: USB SSD mounted at `/mnt/backup`.
- [ ] Daily job + prune; monthly test restore.
- [ ] Back up Docker volumes and `/etc` + `/home`.

---

## 4) FILE SHARING (GUI + EASY)
**Minimum:** Mini PC  
**Recommended:** Mini PC + extra NVMe/USB SSD for space

**Samba (native in Explorer/Finder)**
- [ ] Create `/srv/share` and users. Export as `\\server\share`.

**Syncthing (GUI and cross‑device)**
- [ ] Install on all PCs and phone. Use Tailscale for off‑LAN sync.
- [ ] Folders: `Shared-Docs`, `Photos`.

**Optional web file manager**
- [ ] `filebrowser` in Docker.

---

## 5) DNS SINKHOLE + RECURSOR
**Minimum:** Mini PC  
**Recommended:** Mini PC + router handing out DNS via DHCP

- [ ] Run AdGuard Home.
- [ ] Run Unbound on `127.0.0.1:5335`. Point AdGuard upstream to it.
- [ ] Set DHCP to hand out the server as DNS.

> Note: YouTube app ads mostly bypass DNS. Use uBlock in browsers, AdGuard app on Android, SmartTube on TV, or Premium.

---

## 6) MONITORING
**Minimum:** Mini PC  
**Recommended:** Mini PC + UPS with USB for NUT power stats

- [ ] node‑exporter + cAdvisor → Prometheus.
- [ ] Grafana dashboards.
- [ ] Uptime‑Kuma for ping/HTTP checks.

---

## 7) CENTRAL LOGGING
**Minimum:** Mini PC  
**Recommended:** Mini PC + larger NVMe for log retention

- [ ] Loki + Promtail. Keep 7–14 days hot; prune older.

---

## 8) SECRETS
**Minimum:** Mini PC  
**Recommended:** Mini PC + nightly backup of `/data` to USB SSD

- [ ] Vaultwarden (Bitwarden). Enable 2FA. Nightly backup of `/data`.

---

## 9) MEDIA SERVER
**Minimum:** Mini PC (Intel iGPU), direct‑play clients  
**Recommended:** Mini PC + external storage; later move libraries to a NAS

- [ ] Jellyfin (or Plex/Emby).
- [ ] Prefer direct‑play. If needed, map `/dev/dri` for Intel iGPU.
- [ ] Media at `/srv/media`.

---

## 10) GAME SERVERS
**Minimum:** Mini PC  
**Recommended:** Mini PC + wired Ethernet; 16–32 GB RAM total

**Minecraft Java 1.21.8 Fabric**
- [ ] Docker image: `itzg/minecraft-server`
- [ ] Set `EULA=TRUE`, `TYPE=FABRIC`, `MEMORY=4G`; map `/data`; keep port `25565` internal only.

**Public access without port‑forward**
- [ ] Install playit.gg agent; link token; tunnel to `localhost:25565`.
- [ ] Enable playit autostart; share the playit address with friends.

**Valheim / Terraria**
- [ ] Use maintained Docker images; map volumes; keep ports internal if using playit.

---

## 11) VULNERABILITY SCANS
**Minimum:** Mini PC  
**Recommended:** Mini PC + spare VM target for safe testing

- [ ] Weekly `nmap` against LAN; write report to Markdown.
- [ ] Monthly OpenVAS in a VM/LXC (heavy).

---

## 12) NAS PLAN (PHASE 2)
**Minimum:** Mini PC + single NVMe + USB SSD backups (no redundancy)  
**Recommended:** 2–4 bay NAS (Unraid/TrueNAS) + parity/ZFS + UPS

**Now**
- [ ] Single NVMe + USB SSD backups.

**Later**
- [ ] 2–4 bay NAS (Unraid or TrueNAS). Do ZFS/parity there.
- [ ] Keep this mini for apps only.

---

## 13) REVERSE PROXY + TLS
**Minimum:** Mini PC  
**Recommended:** Mini PC + your own domain for clean internal names

- [ ] Caddy as front door for internal names; SSO on admin apps (see §15).

---

## 14) HOME DASHBOARD
**Minimum:** Mini PC  
**Recommended:** Mini PC + domain‑based URLs for links

- [ ] Homer or Dashy. One page with links, health, notes.

---

## 15) IDENTITY + SSO
**Minimum:** Mini PC  
**Recommended:** Mini PC + hardware 2FA keys for admin accounts

- [ ] Authentik or Authelia in front of key apps.
- [ ] Tailscale ACLs for coarse access; SSO for fine access.

---

## 16) AUTOMATION
**Minimum:** Mini PC  
**Recommended:** Mini PC + private Git repo (Gitea) for IaC

- [ ] Ansible playbooks for installs, updates, backups; store in Git.

---

## 17) CAMERA NVR (OPTIONAL)
**Minimum:** Mini PC + 1–2 IP cams  
**Recommended:** Mini PC + PoE switch + Coral TPU if >2 cams or person detection

- [ ] Frigate + MQTT + Home Assistant. Add Coral TPU if scaling up.

---

## 18) HARDWARE ROADMAP
**Minimum:** Mini PC + USB SSD + UPS  
**Recommended:** 10″ rack, 8–16 port managed switch, separate pfSense box, NAS

- [ ] Today: HP mini, 2×16 GB, 1–2 TB NVMe, USB SSD, UPS.
- [ ] Next: 8–16 port managed switch; small 10″ rack.
- [ ] Later: pfSense box with 2–4 NICs; dedicated NAS.

---

## 19) PORTS + NETWORK NOTES
**Minimum:** Tailscale + playit.gg (no public ports)  
**Recommended:** If exposing anything, add Caddy + SSO + fail2ban

- [ ] Prefer Tailscale and playit. No public port‑forwards.
- [ ] If you ever expose: reverse proxy + SSO + fail2ban; use DNS names.

---

## 20) WEEKLY CARE
**Minimum:** Mini PC  
**Recommended:** Mini PC + cron jobs for updates and backups

- [ ] Update and restart containers:
  ```bash
  docker compose pull && docker compose up -d
  ```
- [ ] Verify backups. Check Grafana and Uptime‑Kuma.
- [ ] Review Loki logs. Clean disk.
