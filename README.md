AI Generated readme, it should cover everyone need to know. 

Recommend to follow trash guide.

# Fedora Server "Arr" Stack (Rootless Podman + Quadlets)

This repository contains a modern, "Fedora-native" configuration for the "Arr" stack (Radarr, Sonarr, Prowlarr, etc.) using **Rootless Podman** and **Systemd Quadlets**. 

Unlike standard Docker Compose setups, this approach integrates directly with Fedora's `systemd` and respects **SELinux** and **User Namespacing** out of the box.

---

## 🚀 Key Features
* **Native Systemd Integration:** Containers are managed as standard systemd services (no more `podman-compose` hacks).
* **Rootless Security:** Containers run entirely within the user's namespace, significantly reducing the attack surface.
* **Samba Compatibility:** Configured to allow a specific host user (e.g., `webuser`) to own the files, making it perfect for NAS setups.
* **SELinux Aware:** Uses proper volume labeling (`:z` and `:Z`) to prevent "Permission Denied" errors without disabling security.

---

## 📂 Directory Structure
Before deploying, ensure your host directories are organized. This setup assumes a unified data structure to support **Atomic Moves** and **Hardlinks**.

```bash
mkdir -p ~/arr-stack/{config/{radarr,sonarr,prowlarr},data/{downloads,media/{movies,tv}}}
```

---

## 🛠️ Deployment Steps

### 1. Enable Lingering
For rootless containers to start automatically on boot (without you being logged in), you **must** enable lingering for your user:
```bash
sudo loginctl enable-linger $USER
```

### 2. Copy the Files
Place the `.container` and `.network` files into the Quadlet directory:
```bash
mkdir -p ~/.config/containers/systemd/
cp ./containers/*.container ~/.config/containers/systemd/
cp ./networks/*.network ~/.config/containers/systemd/
```

### 3. Generate and Start
Tell systemd to "compile" the Quadlets into services:
```bash
systemctl --user daemon-reload
systemctl --user start radarr.service sonarr.service prowlarr.service
```

### 4. Open the Firewall
Fedora's `firewalld` is strict. Open the necessary ports:
```bash
sudo firewall-cmd --permanent --add-port={7878,8989,9696}/tcp
sudo firewall-cmd --reload
```

---

## 🧠 The "Fedora Way" Logic (Read This!)

### Why `PUID=0`?
In most Docker guides, you see `PUID=1000`. In **Rootless Podman**, we use `PUID=0`. 
* **The Logic:** Inside the rootless namespace, "Root" (UID 0) **is** your host user. 
* **The Benefit:** This allows the container's internal initialization scripts to have the "permissions" they need to set up the `/config` folder while physically writing files as your standard host user.

### SELinux Volume Labels
* **`:Z`**: Used for private folders (like `/config`). No other container can touch these.
* **`:z`**: Used for shared folders (like `/data`). This allows Radarr and Sonarr to access the same movie library without conflicts.

### Hardlinks & Atomic Moves
To ensure instant file moves and zero disk overhead, ensure both your downloads and your media library reside on the same filesystem and are mapped under the same root volume (e.g., `/data`).

---

## 📋 Troubleshooting
* **Check Status:** `systemctl --user status radarr.service`
* **View Logs:** `journalctl --user -u radarr.service -f`
* **Syntax Check:** If a service doesn't appear, check for syntax errors:
    `/usr/libexec/podman/quadlet -dryrun -user`

---
