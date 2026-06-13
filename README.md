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
Smart RPC Gateway with Active Failover and Telemetry

This repository contains a production-ready Web3 infrastructure template designed to deploy a highly available, self-healing Smart RPC Gateway. The system sits between decentralized applications (dApps) and various blockchain backends, preventing downtime by automatically routing client traffic away from failing local EVM nodes to secure public endpoints in milliseconds.

The entire stack is containerized and comes with a integrated telemetry suite (Prometheus and Grafana) to measure routing performance, latency, and system load.

Architecture Overview

The system routes user traffic and monitors performance metrics through the following network architecture:

                  [ Client / dApp / cURL ]
                              │
                              ▼ (Port 8888)
                   ┌────────────────────┐
                   │  Smart RPC Proxy   │◀────────┐
                   │     (Nginx)        │         │ Metrics
                   └─────────┬──────────┘         │ Scrape
            Primary Route    │   │ Backup Route   │ (9113)
             (Healthy)       │   │ (Failover 502) │
                             ▼   ▼                │
     ┌───────────────────────┐   ┌────────────┐ ┌─┴──────────────┐
     │ local-evm-blockchain  │   │ Cloudflare │ │ nginx-exporter │
     │    (Anvil Node)       │   │  Web3 Edge │ └───────▲────────┘
     └───────────────────────┘   └────────────┘         │
                                                        │ Scrape (5s)
                                                ┌───────┴────────┐
                                                │   Prometheus   │
                                                └───────▲────────┘
                                                        │ Read
                                                ┌───────┴────────┐
                                                │    Grafana     │
                                                │  (Dashboard)   │
                                                └────────────────┘


Core Design Principles

• Dynamic Name Resolution: The reverse proxy does not crash if the primary blockchain node is down during initialization. It relies on Docker's internal DNS resolver (127.0.0.11) to resolve upstream hosts asynchronously at runtime.

• Transparent Failover: Nginx intercepts connection failures (HTTP 502 and 504) from the primary node. It seamlessly routes failed requests to the backup node using TLS Server Name Indication (SNI) verification to bypass security filters.

• Low-Overhead Telemetery: System stats are exposed by Nginx on an isolated port, translated into Prometheus-compatible metrics, scraped every 5 seconds, and rendered on a dashboard without degrading RPC transit performance.

Local Port Mapping

The environment exposes several local endpoints for development, debugging, and visualization:

• 8888 - Smart RPC Proxy (Nginx)

• 8545 - Local EVM Node (Anvil)

• 8080 - Blockchain Explorer (Otterscan)

• 3000 - Grafana Dashboard

• 9090 - Prometheus Dashboard

Setup and Deployment

Prerequisites

Make sure you have Docker Desktop and the Docker Compose V2 plugin installed on your system.

Running the Stack

Clone the repository and run the services in detached mode:

[bash]
git clone <your-repository-url>
cd web3-infra-stack
docker compose up -d


Verify that all 8 containers are up and running properly:

[bash]
docker compose ps


Resilience Testing (Chaos Engineering)

Follow these steps to observe the gateway's failover and recovery behavior in real-time.

1. Normal State Routing

Send a standard JSON-RPC query to the proxy port. Nginx will route this directly to your fast local Anvil node.

[bash]
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}' -H "Content-Type: application/json" http://localhost:8888


Response: {"jsonrpc":"2.0","id":1,"result":"0x0"}

2. Simulating a Node Crash

Stop the primary blockchain container to simulate an unexpected server outage.

[bash]
docker stop local-evm-blockchain


3. Failover Verification

Send the same query to the proxy. Nginx will instantly detect the local node is down, reroute the request to Cloudflare's public Ethereum gateway, and return the real Ethereum mainnet block number.

[bash]
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}' -H "Content-Type: application/json" http://localhost:8888


4. Self-Healing Recovery

Restart your local node. Nginx automatically resumes routing traffic to the local network as soon as the container responds, with no manual configuration required.

[bash]
docker start local-evm-blockchain


Telemetry Configuration

To visualize your gateway traffic and system health:

1. Open Grafana at http://localhost:3000 (Default credentials: admin / admin).

2. Go to Connections > Data Sources, add Prometheus, and set the connection URL to http://prometheus:9090. Save and test the connection.

3. Go to Dashboards > New > Import, enter the official Nginx Exporter ID 12708, select your Prometheus data source, and load the dashboard.



