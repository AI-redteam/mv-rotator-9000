# wg-rotate-mullvad

A lightweight WireGuard endpoint rotator designed for Mullvad VPN on VPS servers.

This tool:

- Rotates through **multiple WireGuard `.conf` files** (e.g., configs from Mullvad ZIP)
- Uses a **systemd timer** to switch tunnels automatically (default: every 120 seconds)
- Ensures **SSH remains reachable** on the VPS public IP, even while using a full-tunnel VPN
- Requires **no Mullvad app** — uses plain WireGuard (`wg-quick`) for full control

Ideal for VPS setups where you want rotation + privacy **without breaking remote access**.

---

## ✨ Features

- 🚀 Automatic rotation of WireGuard endpoints  
- 🔁 Round-robin config cycling  
- 🛡 SSH-safe: prevents Mullvad from breaking your SSH session  
- 🔧 Simple `wg-quick` backend (no Mullvad firewall interference)  
- 📦 Drop-in scripts and systemd units  
- 🧩 Works with *any* WireGuard configs, not only Mullvad  

---

## 📋 Requirements

- Linux server with systemd (Debian/Ubuntu recommended)  
- WireGuard installed:

```bash
sudo apt update
sudo apt install -y wireguard
```

- Multiple Mullvad WireGuard config files (download ZIP from Mullvad account page)

---

## 📁 Installation

### 1. Clone the repository

```bash
git clone https://github.com/YOURUSER/wg-rotate-mullvad.git
cd wg-rotate-mullvad
```

### 2. Install scripts

```bash
sudo install -Dm755 bin/wg-rotate /usr/local/bin/wg-rotate
sudo install -Dm755 bin/wg-patch-configs /usr/local/bin/wg-patch-configs
```

### 3. Install systemd units

```bash
sudo install -Dm644 systemd/wg-rotate.service /etc/systemd/system/wg-rotate.service
sudo install -Dm644 systemd/wg-rotate.timer   /etc/systemd/system/wg-rotate.timer
```

---

## 🔐 SSH Safety (Required on VPS)

When a VPS uses a full-tunnel WireGuard setup, SSH dies unless you pin replies from your VPS public IP back to the main routing table.

This requires adding the following in every WireGuard config under `[Interface]`:

```
PostUp = ip rule add from <YOUR_VPS_PUBLIC_IP> lookup main
PostDown = ip rule delete from <YOUR_VPS_PUBLIC_IP> lookup main
```

To patch all configs automatically, run:

```bash
sudo wg-patch-configs /etc/wireguard <YOUR_VPS_PUBLIC_IP>
```

---

## 🚀 Setup

### 1. Place all Mullvad configs into `/etc/wireguard`

```bash
sudo mkdir -p /etc/wireguard
sudo cp /path/to/mullvad/*.conf /etc/wireguard/
sudo chmod 600 /etc/wireguard/*.conf
```

### 2. Patch all configs for SSH safety

```bash
sudo wg-patch-configs /etc/wireguard <YOUR_VPS_PUBLIC_IP>
```

### 3. Test manual rotation

```bash
sudo wg-rotate
sudo wg
```

Repeat twice — the interface name / endpoint should change without dropping SSH.

---

## ⏱ Enable Automatic Rotation

Start the timer:

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now wg-rotate.timer
```

Check timer status:

```bash
systemctl status wg-rotate.timer
```

---

## 🔍 Verification

### Check active WireGuard tunnel

```bash
sudo wg
```

### Check SSH safety rule

```bash
ip rule
```

Look for:

```
from <YOUR_VPS_PUBLIC_IP> lookup main
```

### Check external IP

```bash
curl https://am.i.mullvad.net/connected
curl https://ifconfig.me
```

After a rotation, `ifconfig.me` should change while SSH stays connected.

---

## ⚙️ Configuration

### Change rotation interval

Edit:

```
systemd/wg-rotate.timer
```

Set:

```
OnUnitActiveSec=120
```

Reload:

```bash
sudo systemctl daemon-reload
sudo systemctl restart wg-rotate.timer
```

---

## 🛠 Debugging

Show recent rotation logs:

```bash
sudo journalctl -u wg-rotate.service -n 20
```

Show active timer:

```bash
systemctl status wg-rotate.timer
```

Show WG state:

```bash
sudo wg
```

---

## 📝 Notes

- Script cycles configs alphabetically unless you rename them.
- If a config fails to connect, the failure will appear in the journal logs.
- Supports any WireGuard provider, not just Mullvad.

---

## 📄 License

MIT License.
