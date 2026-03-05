# GOLDBRIX Seed Node (Mainnet) — Operator Guide

This document shows how to run a **seed node** (wallet disabled) on Linux.

Ports:
- P2P: 39444/tcp (public)
- RPC: local only (127.0.0.1)

## 1) Download + Verify release
Download the latest release assets:
- goldbrix-v30.0.0-linux-x86_64.tar.gz
- goldbrix-v30.0.0-linux-x86_64.tar.gz.sha256

Verify:
sha256sum -c goldbrix-v30.0.0-linux-x86_64.tar.gz.sha256

Extract:
tar -xzf goldbrix-v30.0.0-linux-x86_64.tar.gz
cd linux-x86_64

## 2) Install binaries
sudo mkdir -p /opt/goldbrix/bin
sudo install -m 0755 bitcoind bitcoin-cli bitcoin-tx /opt/goldbrix/bin/

## 3) Create datadir + config (seed, wallet disabled)
sudo mkdir -p /var/lib/goldbrix/node
sudo tee /var/lib/goldbrix/node/bitcoin.conf >/dev/null <<'EOF'
listen=1
server=1
daemon=0

# network
port=39444

# rpc local only
rpcbind=127.0.0.1
rpcallowip=127.0.0.1
rpcport=8332

# seed nodes should not hold keys
disablewallet=1

# fee (only needed if you broadcast tx from this node; usually not needed for seeds)
fallbackfee=0.0001
EOF

## 4) Systemd service
sudo tee /etc/systemd/system/goldbrix-seed.service >/dev/null <<'EOF'
[Unit]
Description=GOLDBRIX Seed Node (mainnet)
After=network-online.target
Wants=network-online.target

[Service]
User=root
Group=root
Type=simple
ExecStart=/opt/goldbrix/bin/bitcoind -datadir=/var/lib/goldbrix/node
Restart=on-failure
RestartSec=5
LimitNOFILE=1048576

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload
sudo systemctl enable --now goldbrix-seed

## 5) Firewall
Allow P2P only:
sudo ufw allow 22/tcp
sudo ufw allow 39444/tcp
sudo ufw --force enable

## 6) Health checks
sudo systemctl status goldbrix-seed --no-pager -l | head -n 40

/opt/goldbrix/bin/bitcoin-cli -rpcconnect=127.0.0.1 -rpcport=8332 getnetworkinfo | head
/opt/goldbrix/bin/bitcoin-cli -rpcconnect=127.0.0.1 -rpcport=8332 getblockchaininfo | head
/opt/goldbrix/bin/bitcoin-cli -rpcconnect=127.0.0.1 -rpcport=8332 getconnectioncount

## 7) Add peer (optional)
To connect two seeds:
# on seed A:
#  bitcoin-cli ... addnode "<seedB_ip>:39444" "onetry"
