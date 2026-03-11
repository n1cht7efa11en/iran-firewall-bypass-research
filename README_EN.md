# 🔥 Iran Firewall Research — What Works & Why

> Real-world research based on hours of testing during an active internet blackout in Iran (March 2026).
> No theory. No guessing. Only what actually worked — and why.

---

## 🧠 Key Finding (TL;DR)

**The protocol matters less than WHERE the server is located.**

After 40+ failed configs on European servers (VLESS+Reality, VMess, Shadowsocks, etc.) we discovered that **all working configs pointed to servers physically located inside Iran**. The Iranian DPI doesn't need to inspect traffic — during a blackout it simply drops or throttles connections to foreign IPs at the infrastructure level.

---

## 🗺️ How Iran's Firewall Works (Simplified)

```
Your phone
    │
    ▼
[Iranian DPI / Firewall]
    │
    ├─── Foreign IP? → Block / Timeout ❌
    │
    └─── Iranian IP? → Pass through ✅
                            │
                            ▼
                    [Iranian Server]
                            │
                            ▼
                    [Internet / Abroad]
```

During an active blackout the firewall doesn't need to analyze protocol or encryption — it simply kills connections to foreign IPs.

---

## 📊 What We Tested

| Protocol | Server Location | Result | Why |
|----------|----------------|--------|-----|
| VLESS + Reality | Germany (Timeweb) | ❌ Timeout | Foreign IP blocked |
| VMess + TCP | Germany (Timeweb) | ❌ Timeout | Foreign IP blocked |
| Shadowsocks aes-256-gcm | Germany (Timeweb) | ❌ Timeout | Foreign IP blocked |
| VLESS + WS + TLS | Germany (Timeweb) | ❌ Timeout | Foreign IP blocked |
| MTProto | Various | ⚠️ Unstable | New servers added constantly |
| VMess + TCP, no TLS | Iran (Tehran, Soroush Rasanheh) | ✅ Works | Server inside Iran |
| VLESS + HTTP + host myket.ir | Iran (Tehran) | ✅ Works | Server inside Iran |
| VLESS + Reality + SNI digikala.com | Iran (Tehran) | ✅ Works | Server inside Iran |

---

## 🔑 The Pattern That Works

All working configs share these traits:

1. **Server is physically in Iran** — Tehran datacenters (AbrArvan, Soroush Rasanheh, FanAvaran Mihan Mizban)
2. **Non-standard port** — 9901, 2013, 17858, 58203 (not 80/443 which are actively monitored)
3. **No TLS** — paradoxically, TLS fingerprints attract DPI attention more than plain traffic
4. **Iranian domains in Host/SNI** — `myket.ir`, `aparat.com`, `digikala.com`, `telewebion.com` (all whitelisted)

---

## 🌍 IP Address Map from Tested Configs

| IP | Country | Provider | Works from Iran |
|----|---------|----------|----------------|
| `72.56.104.232` | 🇩🇪 Germany | Timeweb | ❌ No |
| `212.16.68.178` | 🇮🇷 Iran | Unknown | ✅ No |
| `151.243.3.59` | 🇳🇱 Netherlands | HOSTKEY B.V. (`hostkey.com`) | ❌ No |
| `87.107.82.152` | 🇮🇷 Iran, Tehran | Soroush Rasanheh | ✅ Yes |
| `185.136.135.202` | 🇮🇷 Iran, Tehran | GeniusMind S.A. | ✅ Yes |
| `188.121.122.40` | 🇮🇷 Iran, Tehran | AbrArvan CDN and IaaS | ✅ Yes |
| `94.184.5.39` | 🇮🇷 Iran | Amir Hosein Maaref (`khalijserver.com`) | ✅ Yes |
| `46.38.150.10` | 🇮🇷 Iran | Peyman Ertebatat Pouya (`farsvps.com`) | ✅ Yes |
| `37.152.179.252` | 🇮🇷 Iran | Unknown | ✅ Yes |
| `2.144.5.9` | 🇮🇷 Iran | Iran Cell Service (`mtnirancell.ir`) — Irancell | ✅ Yes (MTProto) |
| `212.80.18.74` | 🇮🇷 Iran | Asre Pardazesh Amin (`aminidc.com`) | ✅ Yes |

> **Conclusion:** The only non-working European IP was Netherlands (HOSTKEY). All working configs point to servers physically inside Iran. Any European server fails during a blackout (40+ European configs were tested).

---

## 📦 Working Config Examples

### MTProto (Telegram only)
```
tg://proxy?server=2.144.5.9&port=4430&secret=7gnbgVptgqMf2nb4ciMMaddwa2didWlsZC5vcmc
tg://proxy?server=212.80.18.74&port=39090&secret=767cf6f14f461cdd166b54eddb53e1b0
```

### VMess + TCP (no TLS) — Iranian server
```
Server: 87.107.82.152 (Tehran, Iran — Soroush Rasanheh)
Port: 9901
Protocol: VMess
TLS: none
Header: none
UUID: 217a16bb-4348-4361-b66d-82bce154615d
```

### VLESS + HTTP obfuscation — Iranian Host header
```
Server: 94.184.5.39
Port: 58203
Protocol: VLESS
Host: myket.ir (Iranian App Store — always whitelisted)
Header: HTTP
TLS: none
```

### VLESS + HTTP — aparat.com (Iranian YouTube)
```
Server: 46.38.150.10
Port: 443
Protocol: VLESS
Host: aparat.com
Header: HTTP
TLS: none
```

### VLESS + Reality — SNI digikala.com (Iranian Amazon)
```
Server: 37.152.179.252
Port: 443
Protocol: VLESS + Reality
SNI: www.digikala.com
Flow: xtls-rprx-vision
Fingerprint: chrome
```

---

## 🏗️ Iranian Hosting Providers

These are the providers running servers that actually work:

| Provider | Location | Notes |
|----------|----------|-------|
| **AbrArvan** (`arvancloud.ir`) | Tehran | Largest Iranian CDN/IaaS, like Cloudflare for Iran |
| **Soroush Rasanheh** | Tehran | AS214922 |
| **FanAvaran Mihan Mizban** | Tehran | AS214922 |
| **Varesh Cloud** | Tehran | TCI-VareshCloud Cage |
| **Netiran Data Center** | Tehran | AS47430 |

---

## 🌐 Why Iranian Host Headers Work (Domain Fronting)

When a config uses `Host: myket.ir`, the DPI sees:
```
GET / HTTP/1.1
Host: myket.ir   ← DPI reads this and says "OK, Iranian app store, allow"
```

The actual TCP connection goes to the VPN server — but DPI doesn't look deeper.

**Best Iranian domains to use as Host:**
- `myket.ir` — Iranian Google Play, can never be blocked
- `aparat.com` — Iranian YouTube, state-tolerated
- `digikala.com` — Iranian Amazon
- `telewebion.com` — State TV, always whitelisted
- `anten.ir` — Iranian news

---

## 📱 Client Apps

| App | Platform | Notes |
|-----|----------|-------|
| **NPV Tunnel** | Android | Most popular in Iran, supports v2ray configs |
| **v2rayNG** | Android | Free, supports all protocols |
| **Hiddify** | Android/iOS | Easy import, good for beginners |
| **V2Box** | iOS | Free, occasional DNS leaks |
| **Shadowrocket** | iOS | $3, most reliable |

---

## ⚠️ Reality Check

During an active military blackout:
- Iranian infrastructure itself becomes unstable (timeouts even on domestic servers)
- Any config will die eventually — IPs get blacklisted
- The solution is **a pool of configs**, not one perfect config
- Night hours have better connectivity (less active jamming)
- Telegram channels posting fresh configs hourly are more reliable than any single server

---

## 📡 Telegram Channels with Fresh Configs

- `https://t.me/saministamm`
- `t.me/iciou`
- `t.me/Skyportall`

---

## 🤝 Contributing

Found a working pattern? Open a PR. The more data points, the better we understand what bypasses Iranian DPI.

**Format:**
```
Protocol | Server IP/domain | Port | TLS | Header | Host | Works from inside Iran? | Works from outside?
```

---

*Built with frustration, persistence, and 40+ failed attempts. Dedicated to everyone trying to stay connected.*

**Iran Firewall 0 — People 1**
