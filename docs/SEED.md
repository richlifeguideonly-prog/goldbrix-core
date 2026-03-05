# GOLDBRIX — Seed Node (Mainnet)

Seed nodes help the network bootstrap. Recommended settings:
- wallet disabled (`disablewallet=1`)
- RPC local-only (127.0.0.1)
- public P2P port: 39444/tcp

## 1) Download + Verify
Download release assets:
- goldbrix-v30.0.0-linux-x86_64.tar.gz
- goldbrix-v30.0.0-linux-x86_64.tar.gz.sha256

Verify:
sha256sum -c goldbrix-v30.0.0-linux-x86_64.tar.gz.sha256

Extract:
tar -xzf goldbrix-v30.0.0-linux-x86_64.tar.gz
cd linux-x86_64
chmod +x bitcoind bitcoin-cli bitcoin-tx

## 2) Install binaries
sudo mkdir -p /opt/goldbrix/bin
sudo install -m 0755 bitcoind bitcoin-cli bitcoin-tx /opt/goldbrix/bin/

## 3) Datadir + config
sudo mkdir -p /var/lib/goldbrix/node
sudo tee /var/lib/goldbrix/node/bitcoin.conf >/dev/null <<'EOF'
server=1
listen=1
daemon=0

# P2P
port=39444

# RPC (local only)
rpcbind=127.0.0.1
rpcallowip=127.0.0.1
rpcport=8332

# seed mode
disablewallet=1

# optional peers
addnode=89.167.125.117:39444
addnode=89.167.36.203:39444

# safety
fallbackfee=0.0001
EOF

## 4) systemd
sudo tee /etc/systemd/system/goldbrix-seed.service >/dev/null <<'EOF'
[Unit]
Description=GOLDBRIX Seed Node (mainnet)
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
User=root
Group=root
ExecStart=/opt/goldbrix/bin/bitcoind -datadir=/var/lib/goldbrix/node
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload
sudo systemctl enable --now goldbrix-seed

## 5) Firewall
sudo ufw allow 22/tcp
sudo ufw allow 39444/tcp
sudo ufw --force enable

## 6) Checks
/opt/goldbrix/bin/bitcoin-cli -rpcconnect=127.0.0.1 -rpcport=8332 -datadir=/var/lib/goldbrix/node getnetworkinfo | head
/opt/goldbrix/bin/bitcoin-cli -rpcconnect=127.0.0.1 -rpcport=8332 -datadir=/var/lib/goldbrix/node getblockchaininfo | head
/opt/goldbrix/bin/bitcoin-cli -rpcconnect=127.0.0.1 -rpcport=8332 -datadir=/var/lib/goldbrix/node getconnectioncount
