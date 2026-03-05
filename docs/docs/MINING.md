# GOLDBRIX Mining (Mainnet) — Operator Guide

This guide shows how to run a **miner node** (wallet enabled) and mine to an address.
Recommended: run miners separately from seed nodes.

Ports:
- P2P: 39446/tcp (optional public; can be private)
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

## 3) Create datadir + config (miner)
sudo mkdir -p /var/lib/goldbrix/miner
sudo tee /var/lib/goldbrix/miner/bitcoin.conf >/dev/null <<'EOF'
listen=1
server=1
daemon=0

# miner ports (separate from seeds)
port=39446

# rpc local only
rpcbind=127.0.0.1
rpcallowip=127.0.0.1
rpcport=8333

# fees
fallbackfee=0.0001

# optional: connect to your seeds
addnode=89.167.125.117:39444
addnode=89.167.36.203:39444
EOF

## 4) Systemd service
sudo tee /etc/systemd/system/goldbrix-miner.service >/dev/null <<'EOF'
[Unit]
Description=GOLDBRIX Miner Node (mainnet)
After=network-online.target
Wants=network-online.target

[Service]
User=root
Group=root
Type=simple
ExecStart=/opt/goldbrix/bin/bitcoind -datadir=/var/lib/goldbrix/miner
Restart=on-failure
RestartSec=5
LimitNOFILE=1048576

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload
sudo systemctl enable --now goldbrix-miner

## 5) Firewall (minimum)
sudo ufw allow 22/tcp
# if you want this miner reachable as a node:
# sudo ufw allow 39446/tcp
sudo ufw --force enable

## 6) Create wallet + address (v30 rule: simple names)
CLI shortcut:
alias mcli="/opt/goldbrix/bin/bitcoin-cli -rpcconnect=127.0.0.1 -rpcport=8333 -datadir=/var/lib/goldbrix/miner"

Create wallet:
mcli createwallet "miner" false true "" false true

Get address:
mcli -rpcwallet=miner getnewaddress

## 7) Mine blocks (CPU mining via built-in generator)
Mine 1 block to your address:
ADDR="$(mcli -rpcwallet=miner getnewaddress)"
mcli generatetoaddress 1 "$ADDR"

Check:
mcli getblockcount
mcli getblockchaininfo | head
mcli getnetworkinfo | head

## Notes
- This is **private mining**: only you control keys in this wallet.
- Seed nodes should keep `disablewallet=1`.
