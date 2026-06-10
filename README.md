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



