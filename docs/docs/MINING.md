# GOLDBRIX — MINING (Mainnet)

Seeds (bootstrap):
- 89.167.125.117:39444
- 89.167.36.203:39444

## 1) Download + verify (Linux x86_64)
Download Release assets:
- goldbrix-v30.0.0-linux-x86_64.tar.gz
- goldbrix-v30.0.0-linux-x86_64.tar.gz.sha256

Verify + extract:
``bash
sha256sum -c goldbrix-v30.0.0-linux-x86_64.tar.gz.sha256
tar -xzf goldbrix-v30.0.0-linux-x86_64.tar.gz
cd linux-x86_64
chmod +x bitcoind bitcoin-cli bitcoin-tx



2) Install binaries
Bash


sudo mkdir -p /opt/goldbrix/bin
sudo install -m 0755 bitcoind bitcoin-cli bitcoin-tx /opt/goldbrix/bin/



3) Miner node config (separate from seeds)
Bash


sudo mkdir -p /var/lib/goldbrix/miner
sudo tee /var/lib/goldbrix/miner/bitcoin.conf >/dev/null <<'EOF'
server=1
listen=1
daemon=0
txindex=1
port=39446
rpcbind=127.0.0.1
rpcallowip=127.0.0.1
rpcport=8333
fallbackfee=0.0001
addnode=89.167.125.117:39444
addnode=89.167.36.203:39444
EOF


5) systemd (auto-start)
Bash


sudo tee /etc/systemd/system/goldbrix-miner.service >/dev/null <<'EOF'
[Unit]
Description=GOLDBRIX Miner Node (mainnet)
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
User=root
Group=root
ExecStart=/opt/goldbrix/bin/bitcoind -datadir=/var/lib/goldbrix/miner
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
sudo systemctl daemon-reload
sudo systemctl enable --now goldbrix-miner


5) Mine
Bash


alias mcli="/opt/goldbrix/bin/bitcoin-cli -rpcconnect=127.0.0.1 -rpcport=8333 -datadir=/var/lib/goldbrix/miner"

mcli createwallet "miner" true true "" true
ADDR=$(mcli -rpcwallet=miner getnewaddress)
mcli -rpcwallet=miner generatetoaddress 1 "$ADDR"
mcli getblockcount
mcli getconnectioncount
Optional burn-mining (example):
Bash
Code kopieren
BURN="CRsbbtdVnietKLiaaxxDmhJfYkjtuQrqXS"
mcli -rpcwallet=miner generatetoaddress 1 "$BURN"
