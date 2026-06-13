<div align="center">

# Smart RPC Gateway

> Production-ready failover proxy that keeps dApps online when your local node dies — switches to PublicNode in <2s with zero downtime

[![Docker](https://img.shields.io/badge/Docker-Ready-2496ED?logo=docker)](https://github.com/luckyroo13/web3-local-infra/pkgs/container/web3-local-infra)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Last Commit](https://img.shields.io/github/last-commit/luckyroo13/web3-local-infra)](https://github.com/luckyroo13/web3-local-infra)

<img src="assets/failover-demo.gif" alt="Failover demo" width="700" />

*Watch: local Anvil stops → gateway auto-routes to mainnet → local recovers*

</div>

## Why this exists

dApps lose users on every RPC outage. Hosted providers charge per request and rate-limit during congestion. I built this because at FedEx I learned a package that doesn't arrive is lost revenue — the same is true for a JSON-RPC request.

This gateway gives you:
- **Hybrid reliability**: use your fast local node, fall back to public infra automatically
- **Observability by default**: know latency and failover rate before users complain
- **Zero vendor lock-in**: run it on your laptop today, on any cloud tomorrow

## Quick Start (30 seconds)

```bash
git clone [https://github.com/luckyroo13/web3-local-infra](https://github.com/luckyroo13/web3-local-infra) && cd web3-local-infra
docker compose up -d
curl -X POST http://localhost:8888 -H "Content-Type: application/json" -d '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}'
Open http://localhost:3000 (admin/admin) for Grafana.
Features
Feature	What it solves
Transparent Failover	Intercepts 502/504 from primary, reroutes to PublicNode securely
Dynamic DNS Resolution	Nginx uses Docker DNS & Google DNS (127.0.0.11 / 8.8.8.8) so proxy starts even if Anvil is down
Low-overhead Telemetry	Nginx exporter → Prometheus scrape every 5s → Grafana dashboard 12708
Self-healing	When Anvil returns, traffic automatically resumes without restart
Full Local Stack	Anvil + Otterscan + Prometheus + Grafana + cAdvisor in one compose
Architecture
Plaintext
Client → :8888 Nginx → Primary: Anvil :8545
                   ↘ Backup: ethereum-rpc.publicnode.com:443 (on failure)
Metrics: Nginx :8080 → exporter :9113 → Prometheus :9090 → Grafana :3000
Proof of Work Checklist
Hiring managers look for this:
[x] Deployed: works locally with real endpoints
[x] Monitoring: Prometheus + Grafana dashboards
[x] Documented: this README with architecture and decisions
[x] Public: real commit history
[ ] CI/CD: GitHub Actions for build/test (in progress)
[ ] IaC: Terraform for reproducible deploy (in progress)
Local Ports
Port	Service
8888	Smart RPC Proxy
8545	Anvil EVM
8080	Otterscan Explorer
3000	Grafana
9090	Prometheus
8081	cAdvisor
9113	Nginx Exporter
Resilience Testing
Normal: curl to :8888 returns "result":"0x0" from Anvil
Crash: docker stop local-evm-blockchain
Failover: same curl returns real mainnet block from PublicNode
Recovery: docker start local-evm-blockchain → traffic returns to local automatically
Telemetry Setup
Grafana → http://localhost:3000 (admin/admin)
Add data source: http://prometheus:9090
Import dashboard ID 12708 (Nginx Prometheus)
Roadmap
[ ] GitHub Actions: lint, build, test compose
[ ] k3s manifests for cheap K8s demo
[ ] Terraform module (VPC + EC2) for optional cloud deploy
[ ] Alertmanager → Slack alerts on failover
Contributing
PRs welcome. Open an issue first for major changes.
License
MIT — see LICENSE