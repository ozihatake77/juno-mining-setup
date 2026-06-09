# 🪙 JUNO Mining Setup Guide

> CPU Mining JUNO (RandomX) via junopool.com

## 📋 Overview

- **Coin:** JUNO (ticker: JUNO)
- **Algorithm:** RandomX (rx/juno)
- **Pool:** [junopool.com](https://junopool.com) (fee 1%, auto payout, 648+ miners)
- **Miner:** junorig (RandomX CPU miner)
- **Best for:** CPU mining (AMD EPYC, Intel Xeon, atau CPU modern lainnya)

---

## 🖥️ VPS Requirements

| Spec | Minimum | Recommended |
|------|---------|-------------|
| CPU | 2 cores | 4-8 cores |
| RAM | 2 GB | 4-8 GB |
| OS | Ubuntu 20.04+ | Ubuntu 22.04 |
| Storage | 10 GB | 20 GB |

> **Note:** RandomX butuh RAM minimal 2GB per instance. Kalau VPS RAM-nya 8GB, bisa jalan 2-4 instance.

---

## 🚀 Quick Setup (One Command)

```bash
# Login ke VPS via SSH
ssh root@YOUR_VPS_IP

# Download & install junorig
curl -L https://github.com/juno-network/junorig/releases/latest/download/junorig-linux-amd64 -o /usr/local/bin/junorig
chmod +x /usr/local/bin/junorig

# Verify installation
junorig --version
```

---

## ⚙️ Configuration

### 1. Buat Wallet Address

Kamu butuh wallet address JUNO untuk menerima hasil mining. Bisa pakai:
- [Keplr Wallet](https://www.keplr.app/) (recommended)
- [Leap Wallet](https://leapwallet.io/)

Setelah punya wallet, catat address-nya (format: `juno1...`)

### 2. Buat Config File

```bash
mkdir -p /etc/junorig
nano /etc/junorig/config.json
```

Isi config:

```json
{
  "pools": [
    {
      "url": "stratum+tcp://pool.junopool.com:3636",
      "user": "JUNO_WALLET_ADDRESS",
      "pass": "x"
    }
  ],
  "cpu": true,
  "threads": 0,
  "background": false,
  "log-file": "/var/log/junorig.log",
  "donate-level": 1
}
```

**Ganti:**
- `JUNO_WALLET_ADDRESS` → wallet address JUNO lo (contoh: `juno1abc123...`)
- `threads` → `0` = auto-detect semua core, atau set manual (contoh: `4`)

### 3. Simpan & keluar

```
Ctrl+X → Y → Enter
```

---

## 🏃 Running the Miner

### Option 1: Manual (Testing)

```bash
junorig --url=stratum+tcp://pool.junopool.com:3636 --user=JUNO_WALLET_ADDRESS --pass=x
```

### Option 2: Background dengan tmux (Recommended)

```bash
# Buat session tmux
tmux new -s mining

# Jalankan miner
junorig --url=stratum+tcp://pool.junopool.com:3636 --user=JUNO_WALLET_ADDRESS --pass=x

# Keluar dari tmux (miner tetap jalan): Ctrl+B → D

# Balik ke session:
tmux attach -t mining
```

### Option 3: Systemd Service (Auto-start on boot)

```bash
cat > /etc/systemd/system/junorig.service << 'EOF'
[Unit]
Description=JUNO RandomX Miner
After=network.target

[Service]
Type=simple
ExecStart=/usr/local/bin/junorig --url=stratum+tcp://pool.junopool.com:3636 --user=JUNO_WALLET_ADDRESS --pass=x
Restart=always
RestartSec=10
Nice=19
CPUQuota=80%

[Install]
WantedBy=multi-user.target
EOF

# Enable & start
systemctl daemon-reload
systemctl enable junorig
systemctl start junorig

# Check status
systemctl status junorig

# View logs
journalctl -u junorig -f
```

---

## 📊 Monitoring

### Cek Status Mining di Pool

Buka: `https://junopool.com` → masukkan wallet address → lihat:
- Hashrate
- Shares accepted/rejected
- Balance
- Payout history

### Cek dari Terminal

```bash
# Lihat log miner
tail -f /var/log/junorig.log

# Atau kalau pakai systemd
journalctl -u junorig -f

# Cek CPU usage
htop

# Cek miner process
ps aux | grep junorig
```

### Contoh Output Miner

```
[2026-06-09 18:30:15] CPU #0: 525.3 H/s
[2026-06-09 18:30:15] CPU #1: 520.1 H/s
[2026-06-09 18:30:15] CPU #2: 518.7 H/s
[2026-06-09 18:30:15] CPU #3: 522.4 H/s
[2026-06-09 18:30:15] Total: 2086.5 H/s
[2026-06-09 18:30:16] Accepted share 1/1 (diff 1234)
```

---

## 💰 Expected Earnings

| VPS Spec | Hashrate | Est. JUNO/day |
|----------|----------|---------------|
| 2 core (basic) | ~1,000 H/s | ~0.02-0.05 |
| 4 core | ~2,000 H/s | ~0.04-0.10 |
| 8 core (EPYC) | ~4,000 H/s | ~0.08-0.20 |

> **Note:** Earnings tergantung difficulty network dan jumlah miner di pool. Ini estimasi kasar.

---

## 🔧 Troubleshooting

### Miner nggak mau jalan
```bash
# Cek apakah junorig ada
which junorig

# Cek permission
ls -la /usr/local/bin/junorig

# Cek error log
tail -20 /var/log/junorig.log
```

### Hashrate rendah
```bash
# Cek CPU governor
cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_governor

# Set ke performance
echo performance | tee /sys/devices/system/cpu/cpu*/cpufreq/scaling_governor
```

### Pool connection error
```bash
# Test koneksi ke pool
nc -zv pool.junopool.com 3636

# Cek firewall
ufw status
ufw allow out 3636
```

### Auto-restart kalau crash
Kalau pakai tmux, bikin script wrapper:

```bash
cat > /root/start_mining.sh << 'EOF'
#!/bin/bash
while true; do
    echo "[$(date)] Starting junorig..."
    junorig --url=stratum+tcp://pool.junopool.com:3636 --user=JUNO_WALLET_ADDRESS --pass=x
    echo "[$(date)] junorig exited. Restarting in 10s..."
    sleep 10
done
EOF
chmod +x /root/start_mining.sh

# Jalankan
tmux new -s mining '/root/start_mining.sh'
```

---

## 📝 Pool Info

| Info | Detail |
|------|--------|
| Pool URL | `stratum+tcp://pool.junopool.com:3636` |
| Fee | 1% |
| Min Payout | Auto |
| Payout | Auto (langsung ke wallet) |
| Miners | 648+ |
| Dashboard | https://junopool.com |

---

## ⚠️ Important Notes

1. **Jangan share wallet address** — itu public, tapi jangan share private key / seed phrase
2. **Monitor VPS cost** — pastikan hasil mining > biaya VPS
3. **CPU usage** — set `CPUQuota=80%` supaya VPS tetap responsif
4. **Backup wallet** — simpan seed phrase di tempat aman (offline)

---

## 📜 License

This guide is for educational purposes. Mining cryptocurrency may have legal and tax implications in your jurisdiction.
