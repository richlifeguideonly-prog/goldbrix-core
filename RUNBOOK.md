# GOLDBRIX Core — Runbook (Mainnet)

## Release assets
Download from GitHub Releases:
- goldbrix-v30.0.0-linux-x86_64.tar.gz
- goldbrix-v30.0.0-linux-x86_64.tar.gz.sha256

Optional:
- RUNBOOK.md (this file)
- docs/SEED.md, docs/FULLNODE.md, docs/MINING.md

## 1) Verify
sha256sum -c goldbrix-v30.0.0-linux-x86_64.tar.gz.sha256

## 2) Extract
tar -xzf goldbrix-v30.0.0-linux-x86_64.tar.gz
cd linux-x86_64
chmod +x bitcoind bitcoin-cli bitcoin-tx

## 3) Choose what to run
- Seed node (wallet disabled): docs/SEED.md
- Full node: docs/FULLNODE.md
- Miner node (wallet enabled, separate ports): docs/MINING.md

## Seed IPs (bootstrap)
- 89.167.125.117:39444
- 89.167.36.203:39444

## Notes
- Seed nodes should run with `disablewallet=1`.
- Keep RPC local-only (127.0.0.1).
