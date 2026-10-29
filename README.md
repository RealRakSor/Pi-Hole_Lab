# pihole-home
This is my first project. The goal is to configure Pi-hole inside a Docker container to cover my entire local network.

## 🗺️ Roadmap: Pi-hole Homelab with Docker & OpenDNS

This roadmap outlines the steps to build, secure, and document a centralized home network filter.

### 🛠️ Step 1: Preparation & Cleanup
- [ ] **Remove old installations:** Uninstall the old, bare-metal Pi-hole installation from your Raspberry Pi to free up ports (especially port 53).
- [ ] **Set a static IP:** Log into your router under *IP Reservation / DHCP Reservation* and lock down your Raspberry Pi's IP address so it never changes.

### 🐳 Step 2: Install Docker & Pi-hole Container
- [ ] **Install Docker and Docker Compose** on your Raspberry Pi via SSH (`cmd`).
- [ ] **Create a `docker-compose.yml` file** for Pi-hole (making it easy to manage and restart the container).
- [ ] **Start the container** and verify that the web interface works.

### 🌐 Step 3: Configure Upstream & Network (OpenDNS)
- [ ] **Select OpenDNS as Upstream** in Pi-hole's settings.
- [ ] **Point your router's LAN-DNS** to your Raspberry Pi's static IP address so that your entire network (including guests) is protected automatically.

---

## 🔍 Network Traffic Flow (Architecture)

Once everything is up and running, the DNS traffic moves through the following chain:
```text
[All devices in the house / Guests] 
       │ (requests web addresses)
       ▼
[Your Router (DHCP Server)] 
       │ (automatically tells devices: "Ask my Raspberry Pi!")
       ▼
[Raspberry Pi / Pi-hole Container] 
       │ (Checks against your blocklists)
       ├─► If ad/malicious ──► Stop! 🛑 (Blocked immediately)
       └─► If allowed ───────► Forwards upstream ──► [OpenDNS] 🌐
