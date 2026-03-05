# GOLDBRIX Full Node (Mainnet) — Operator Guide

This guide shows how to run a full node (wallet optional).  
Recommended: keep RPC local-only.

Ports:
- P2P: 39444/tcp (public)
- RPC: local only (127.0.0.1)

## 1) Download + Verify release
Download:
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

## 3) Create datadir + config
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

# optional: connect to seeds
addnode=89.167.125.117:39444
addnode=89.167.36.203:39444

# fee (needed only if you broadcast tx from this node)
fallbackfee=0.0001
EOF

## 4) Systemd service
sudo tee /etc/systemd/system/goldbrix-node.service >/dev/null <<'EOF'
[Unit]
Description=GOLDBRIX Full Node (mainnet)
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
sudo systemctl enable --now goldbrix-node

## 5) Firewall
sudo ufw allow 22/tcp
sudo ufw allow 39444/tcp
sudo ufw --force enable

## 6) Health checks
alias ncli="/opt/goldbrix/bin/bitcoin-cli -rpcconnect=127.0.0.1 -rpcport=8332 -datadir=/var/lib/goldbrix/node"

ncli getconnectioncount
ncli getblockchaininfo | head
ncli getnetworkinfo | head
