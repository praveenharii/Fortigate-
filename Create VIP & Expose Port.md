# FortiGate — Port Forwarding Port 1433 (SQL Server)

## Overview

This documents the configuration to expose SQL Server port `1433` publicly through a FortiGate firewall sitting behind a double NAT setup (TM UniFi ZTE ZXHN F6600P router).

**Topology:**
```
Internet (Public IP: xxx.xxx.xxx.xxx)
        ↓
  ZTE Router (PPPoE, NAT, LAN: 192.168.0.1)
        ↓
  FortiGate WAN (Static: 192.168.0.5) ← DMZ target
        ↓
  FortiGate LAN (192.168.1.xxx)
        ↓
  SQL Server (192.168.1.x, port 1433)
```

---

## Prerequisites

- DMZ enabled on ZTE router pointing to `192.168.0.5`
- FortiGate WAN interface set to static IP `192.168.0.5`

---

## Step 1 — Create Virtual IP (VIP)

**GUI:** Policy & Objects → Virtual IPs → Create New → Virtual IP

| Field | Value |
|---|---|
| Name | `VIP-SQL-1433` |
| Interface | `wan1` |
| External IP Address | `0.0.0.0` |
| Map to IPv4 Address | `192.168.1.x` ← SQL Server LAN IP |
| Enable Port Forwarding | ✅ Yes |
| Protocol | TCP |
| External Service Port | `1433` |
| Map to IPv4 Port | `1433` |

**CLI:**
```bash
config firewall vip
    edit "VIP-SQL-1433"
        set interface "wan1"
        set extip 0.0.0.0
        set extport 1433
        set mappedip 192.168.1.x
        set mappedport 1433
        set portforward enable
        set protocol tcp
    next
end
```

---

## Step 2 — Create Firewall Policy

**GUI:** Policy & Objects → Firewall Policy → Create New

| Field | Value |
|---|---|
| Name | `Allow-SQL-1433` |
| Incoming Interface | `wan1` |
| Outgoing Interface | `lan` |
| Source | `all` (restrict to trusted IPs in production) |
| Destination | `VIP-SQL-1433` |
| Service | `MS-SQL` or custom port `1433` |
| Action | Accept |
| NAT | ✅ Enabled |
| Status | ✅ Enabled |

**CLI:**
```bash
config firewall policy
    edit 0
        set name "Allow-SQL-1433"
        set srcintf "wan1"
        set dstintf "lan"
        set srcaddr "all"
        set dstaddr "VIP-SQL-1433"
        set service "MS-SQL"
        set action accept
        set nat enable
        set status enable
    next
end
```

---

## Step 3 — Verify

Test from an external network (mobile data) using an online port checker:
- Host: `xxx.xxx.xxx.xxx`
- Port: `1433`
- Expected result: **Open**

---

## Security Recommendations

> ⚠️ Port 1433 is one of the most targeted ports on the internet. Do not leave it open to all sources.

- Restrict **Source** in the firewall policy to **trusted IPs only** instead of `all`
- Enable an **IPS profile** on the firewall policy to block SQL brute force attempts
- Keep SQL Server **patched and up to date**
- Consider using **SSL-VPN** instead of direct port exposure for better security
