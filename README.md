# 🪙 JUNO Mining Setup Guide

> CPU Mining JUNO (RandomX) via junopool.com

## 📋 Overview

- **Coin:** JUNO (ticker: JUNO)
- **Algorithm:** RandomX (rx/juno)
- **Pool:** [junopool.com](https://junopool.com) (fee 1%, auto payout, 648+ miners)
- **Miner:** junorig (RandomX CPU miner)
- **Best for:** CPU mining (AMD EPYC, Intel Xeon, or any modern CPU)

---

## 💳 Create a Wallet

Before mining, you need a JUNO wallet to receive your rewards.

**Go to:** [thejunowallet.com](https://thejunowallet.com/)

- Works on **PC (desktop browser)** and **Mobile (phone browser)**
- Install the wallet extension or use the web version
- Create a new wallet and **save your seed phrase in a safe place (offline)**
- Copy your wallet address (format: `juno1...`)

> ⚠️ **NEVER share your seed phrase or private key with anyone.**

---

## 🖥️ VPS Requirements

| Spec | Minimum | Recommended |
|------|---------|-------------|
| CPU | 2 cores | 4-8 cores |
| RAM | 2 GB | 4-8 GB |
| OS | Ubuntu 20.04+ | Ubuntu 22.04 |
| Storage | 10 GB | 20 GB |

> **Note:** RandomX requires at least 2GB RAM per instance. With 8GB RAM, you can run 2-4 instances.

---

## 🚀 Quick Setup (One Command)

```bash
# Login to your VPS via SSH
ssh root@YOUR_VPS_IP

# Download & install junorig
curl -L https://github.com/juno-network/junorig/releases/latest/download/junorig-linux-amd64 -o /usr/local/bin/junorig
chmod +x /usr/local/bin/junorig

# Verify installation
junorig --version
```

---

## ⚙️ Configuration

### 1. Create Config File

```bash
mkdir -p /etc/junorig
nano /etc/junorig/config.json
```

Paste this config:

```json
{
  "pools": [
    {
      "url": "stratum+tcp://pool.junopool.com:3636",
      "user": "YOUR_JUNO_WALLET_ADDRESS",
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

**Replace:**
- `YOUR_JUNO_WALLET_ADDRESS` → your JUNO wallet address (e.g. `juno1abc123...`)
- `threads` → `0` = auto-detect all cores, or set manually (e.g. `4`)

### 2. Save & Exit

```
Ctrl+X → Y → Enter
```

---

## 🏃 Running the Miner

### Option 1: Manual (For Testing)

```bash
junorig --url=stratum+tcp://pool.junopool.com:3636 --user=YOUR_JUNO_WALLET_ADDRESS --pass=x
```

### Option 2: Background with tmux (Recommended)

```bash
# Create a tmux session
tmux new -s mining

# Run the miner
junorig --url=stratum+tcp://pool.junopool.com:3636 --user=YOUR_JUNO_WALLET_ADDRESS --pass=x

# Detach from tmux (miner keeps running): Ctrl+B → D

# Re-attach to session:
tmux attach -t mining
```

### Option 3: Systemd Service (Auto-start on Boot)

```bash
cat > /etc/systemd/system/junorig.service << 'EOF'
[Unit]
Description=JUNO RandomX Miner
After=network.target

[Service]
Type=simple
ExecStart=/usr/local/bin/junorig --url=stratum+tcp://pool.junopool.com:3636 --user=YOUR_JUNO_WALLET_ADDRESS --pass=x
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

# View live logs
journalctl -u junorig -f
```

---

## 📊 Monitoring

### Check Mining Status on Pool

Open: `https://junopool.com` → enter your wallet address → view:
- Hashrate
- Shares accepted / rejected
- Balance
- Payout history

### Check from Terminal

```bash
# View miner logs
tail -f /var/log/junorig.log

# Or if using systemd
journalctl -u junorig -f

# Check CPU usage
htop

# Check miner process
ps aux | grep junorig
```

### Example Miner Output

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
| 2 cores (basic) | ~1,000 H/s | ~0.02-0.05 |
| 4 cores | ~2,000 H/s | ~0.04-0.10 |
| 8 cores (EPYC) | ~4,000 H/s | ~0.08-0.20 |

> **Note:** Earnings depend on network difficulty and total pool hashrate. These are rough estimates.

---

## 🔧 Troubleshooting

### Miner won't start
```bash
# Check if junorig exists
which junorig

# Check permissions
ls -la /usr/local/bin/junorig

# Check error logs
tail -20 /var/log/junorig.log
```

### Low hashrate
```bash
# Check CPU governor
cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_governor

# Set to performance mode
echo performance | tee /sys/devices/system/cpu/cpu*/cpufreq/scaling_governor
```

### Pool connection error
```bash
# Test connection to pool
nc -zv pool.junopool.com 3636

# Check firewall
ufw status
ufw allow out 3636
```

### Auto-restart on crash
Create a wrapper script:

```bash
cat > /root/start_mining.sh << 'EOF'
#!/bin/bash
while true; do
    echo "[$(date)] Starting junorig..."
    junorig --url=stratum+tcp://pool.junopool.com:3636 --user=YOUR_JUNO_WALLET_ADDRESS --pass=x
    echo "[$(date)] junorig exited. Restarting in 10s..."
    sleep 10
done
EOF
chmod +x /root/start_mining.sh

# Run it
tmux new -s mining '/root/start_mining.sh'
```

---

## 📝 Pool Info

| Info | Detail |
|------|--------|
| Pool URL | `stratum+tcp://pool.junopool.com:3636` |
| Fee | 1% |
| Min Payout | Auto |
| Payout | Auto (direct to wallet) |
| Active Miners | 648+ |
| Dashboard | https://junopool.com |

---

## ⚠️ Important Notes

1. **Never share your seed phrase or private key**
2. **Monitor VPS costs** — make sure mining rewards > VPS cost
3. **CPU usage** — set `CPUQuota=80%` to keep VPS responsive
4. **Backup your wallet** — store seed phrase offline in a safe place

---

## 📜 License

This guide is for educational purposes. Mining cryptocurrency may have legal and tax implications in your jurisdiction.
