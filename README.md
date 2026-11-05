# pihole-home
This is my first project. The goal is to configure Pi-hole inside a Docker container to cover my entire local network.

## 🎯 Project Roadmap & Status

- [x] **Milestone 1: Host & Network Setup**
  * Clean installation of Raspberry Pi OS with a static IP via router DHCP reservation.
  * Installed Git directly on the host for native configuration management.
  * Audited port 53 availability (`netstat -tulpn`) and verified initial gateway DNS routing.

- [x] **Milestone 2: Docker & Bridge Architecture**
  * Configured Docker with explicit port mappings (`53:53`, `80:80`, `443:443`) in **bridge mode** for container isolation and future scalability.
  * Created a structured custom configuration (`docker-compose-pihole.yml`) managed via explicit file flags (`-f`).

- [ ] **Milestone 3: Web UI & Upstream Configuration** *(Current)*
  * Verify admin access via browser (`http://<PI_IP>/admin`).
  * Integrate OpenDNS as the upstream DNS provider for enhanced network filtering.

- [ ] **Milestone 4: Network-Wide Integration**
  * Configure router DHCP settings to point all local network devices to the Pi-hole DNS server.
  * Monitor query logs and verify ad-blocking functionality across client devices.

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
```
## 🏆 Completed Milestones & Implementation Guide

### Milestone 1: Host & Network Setup
* **What was done:** Prepared the physical Raspberry Pi environment, established reliable network routing, and set up native version control.
* **How it was done:**
  * **OS & Static IP:** Performed a clean installation of Raspberry Pi OS and configured a permanent static IP address via the home router's DHCP/IP reservation to ensure stable server reachability.
  * **Host-Level Version Control:** Installed Git directly on the Raspberry Pi host OS (`sudo apt install git`) to maintain native control over project documentation and infrastructure configuration files.
  * **Network & Port Auditing:** Inspected system DNS behavior (`/etc/resolv.conf`) and verified port 53 availability (`netstat -tulpn`), confirming a clean environment ready for network-wide DNS interception.

### Milestone 2: Containerization & Docker Architecture
* **What was done:** Deployed Pi-hole inside an isolated container using a clean, scalable, and structured Docker Compose workflow.
* **How it was done:**
  * **Docker Engine & Permissions:** Installed Docker via the official installation script and configured user daemon privileges (`sudo usermod -aG docker $USER`) for secure command execution.
  * **Bridge Mode & Port Isolation:** Deliberately bypassed `network_mode: host` in favor of standard **bridge mode** with explicit port mappings (`53:53/tcp+udp`, `80:80/tcp`, `443:443/tcp`). This isolates the Pi-hole service and preserves host ports for future homelab containers.
  * **Custom Configuration Management:** Created a dedicated configuration file named `docker-compose-pihle.yml` to maintain clean multi-container directory organization.
  * **Detached Deployment:** Started the stack in the background using the explicit file flag:
    ```bash
    docker compose -f docker-compose-pihle.yml up -d
    ```
---

## 📌 Repository Structure & GitHub Integration

### 1. Git & SSH Authentication on Raspberry Pi
- **Git Identity:** Configured global `user.name` and `user.email` directly in the terminal on the Raspberry Pi.
- **SSH Key (Ed25519):** Generated an Ed25519 key pair (`~/.ssh/id_ed25519`) and added the public key to GitHub for secure, passwordless authentication.
- **Connection Verification:** Tested and verified the SSH connection against GitHub using `ssh -T git@github.com`.

### 2. Project Structure & Isolation
- **Dedicated Directory:** Created the project folder `~/Pi-Hole_Lab/` to isolate project configuration files from the home directory (`~`).
- **File Organization:** Moved `docker-compose-pihle.yml` into the dedicated project workspace.
- **Remote Link:** Initialized the directory as a local Git repository and linked it to the remote repository (`git@github.com:RealRakSor/Pi-Hole_Lab.git`).

### 3. Security & Secret Management
- **Environment Variables in Compose:** Updated `docker-compose-pihle.yml` so that sensitive parameters (such as `FTLCONF_webserver_api_password`) utilize variable interpolation (`${PIHOLE_PASSWORD}`).
- **Local `.env` File:** Created a local `.env` file inside `~/Pi-Hole_Lab/` containing the actual plaintext password, keeping it strictly on the Raspberry Pi host.
- **`.gitignore` Exclusion:** Implemented a `.gitignore` file to ensure `.env` and local data directories are omitted from version control, preventing accidental credential leaks on GitHub.
- **Configuration Verification:** Validated variable parsing and compose syntax locally using `docker compose -f docker-compose-pihle.yml config`.

### 4. Version Control & GitHub Push
- **Remote Synchronization:** Resolved initial push rejections (`fetch first` / `unrelated histories`) caused by remote initialization files by syncing remote changes (`git pull`).
- **Successful Push:** Staged, committed, and pushed the infrastructure blueprints (`docker-compose-pihle.yml`) and exclusion rules (`.gitignore`) to the `main` branch.
- **Repository Status:** Verified a clean working tree (`working tree clean`) with the local environment fully synchronized with `origin/main`.

---
    ## 🛠️ Technical Troubleshooting & Practical Learnings

### 1. SSH Host Key Fingerprint Update
* **The Issue:** Host identification mismatch warning (`REMOTE HOST IDENTIFICATION HAS CHANGED!`) after a clean OS reinstallation on the existing static IP.
* **The Solution:** Manually updated the client-side `known_hosts` file with the new server fingerprint to re-establish trust.

### 2. Secret Management & Environment Separation (`.env`)
* **The Lesson:** Hardcoding sensitive values (like the Pi-hole web password) directly into configuration files poses a major security risk for public repositories.
* **Implementation:** Separated infrastructure from secrets by utilizing an environment file (`.env`) with variable interpolation (`${PIHOLE_PASSWORD}`) while ensuring `.env` is excluded via `.gitignore`.

### 3. Environment File Context (`.env` Placement)
* **The Issue:** Docker warning: `The "PIHOLE_PASSWORD" variable is not set`.
* **The Cause:** The `.env` file was placed in the home directory (`~`) rather than side-by-side with the target `docker-compose-pihle.yml` file.
* **The Solution:** Ensured all environment configuration files reside in the exact same directory as the executing compose file, as Docker looks locally for context.

### 4. Git Staging & File Existence
* **The Issue:** `fatal: pathspec '.gitignore' did not match any files`.
* **The Cause:** Attempted to stage a file (`git add`) before it was physically created on the disk.
* **The Solution:** Always create or write files to the filesystem first (e.g., via `nano` or shell commands) before attempting to track them with Git.

### 5. Remote Repository Synchronization (`fetch first` / Rejected Push)
* **The Issue:** `git push` rejected with a `fetch first` error.
* **The Cause:** The remote GitHub repository was initialized with a default file (such as `README.md`), causing the remote history to diverge from the local repository.
* **The Solution:** Synchronized remote changes to the local environment (`git pull`) before pushing local commits.

### 6. Host OS vs. Container Architecture Boundaries
* **The Lesson:** Established a clear separation of concerns in the homelab environment:
  * **Host OS (Raspberry Pi):** The control plane where source code, Git repositories, SSH keys, and Docker Compose blueprints reside.
  * **Containers (Docker):** Ephemeral, isolated runtimes dedicated strictly to executing the application (Pi-hole) without hosting development tools or credential stores.
