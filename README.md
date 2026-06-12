# Automated High-Availability Edge-Computing & Secure Proxy Tunneling Pipeline

## The Architecture (The Flex)

This repository contains a battle-tested infrastructure for running a highly optimized, passive income node farm and heavy compute workloads on a constrained 4GB VPS.

### Distributed Edge Computing
Running multiple lightweight bandwidth daemons concurrently (Traffmonetizer, Repocket, PacketStream, GagaNode, Titan, Bitping) to maximize passive income generation across various decentralized networks.

### Strict Network Namespace Isolation
Specific nodes are forced through dedicated proxy tunnels. By leveraging Docker's `network_mode: service:[tunnel_container]`, we bind the edge node's network namespace directly to a SOCKS5/HTTP proxy tunnel (managed by `tun2socks`). If the proxy drops, the node loses connection instantly—preventing IP leaks and ensuring datacenter IPs aren't inadvertently banned.

### Resource Governance
Orchestrating heavy compute containers (like Chromium) alongside latency-sensitive bandwidth nodes without experiencing Out-Of-Memory (OOM) kills. This is achieved via strict Docker Compose deployment resource limits (`cpus` and `memory` reservations/limits), ensuring the compute layers are throttled before they can starve the network daemons or crash the host system.

## Architecture Diagram

```text
                           +------------------------+
                           |  Internet / Datacenter |
                           +-----------+------------+
                                       |
+--------------------------------------|---------------------------------------+
| Docker Host (4GB VPS)                |                                       |
|                                      v                                       |
|  +-----------------------+     +-------------------+     +-----------------+ |
|  | THE HEAVY COMPUTE     |     | THE DIRECT NODES  |     | THE PROXY LAYER | |
|  |                       |     |                   |     |                 | |
|  | +-------------------+ |     | +---------------+ |     | +-------------+ | |
|  | |   Chromium (UI)   | |<--->| |  Titan Edge   | |<--->| | tun2socks   | | |
|  | | [Resource Capped] | |     | +---------------+ |     | | (VPN/Proxy) | | |
|  | +-------------------+ |     | +---------------+ |     | +-------------+ | |
|  +-----------------------+     | |   Bitping     | |     |       ^         | |
|                                | +---------------+ |     +-------|---------+ |
|                                +-------------------+             |           |
|                                                                  v           |
|                                                      +---------------------+ |
|                                                      | THE EDGE NODES      | |
|                                                      | (Network Isolated)  | |
|                                                      |                     | |
|                                                      | +-----------------+ | |
|                                                      | | Traffmonetizer  | | |
|                                                      | | Repocket        | | |
|                                                      | | PacketStream    | | |
|                                                      | | GagaNode        | | |
|                                                      | +-----------------+ | |
|                                                      +---------------------+ |
+------------------------------------------------------------------------------+
```
*(Traffic from The Edge Nodes is strictly routed into the tun2socks network namespace, which then exits to the internet via the designated proxy.)*

## Deployment & Setup

### Prerequisites
Ensure your remote host meets the following requirements:
- **OS**: Linux-based distribution (Ubuntu 20.04/22.04 LTS or Debian recommended)
- **Engine**: Docker Engine v20.10+ and Docker Compose v2.0+ installed
- **Hardware**: Minimum 1 vCPU and 2GB RAM (Optimized here for a 4GB VPS boundary)

### Step-by-Step Installation

#### 1. Clone the Infrastructure Repository
Pull the deployment framework onto your remote machine:
```bash
git clone https://github.com/Jer123-dev/edge-compute-proxy-pipeline.git
cd edge-compute-proxy-pipeline
```

#### 2. Environment Configuration
The architecture strictly separates core infrastructure design from runtime secrets. Populate the deployment environment variables before initializing the containers:
```bash
# Copy the template to a live environment file
cp .env.example .env

# Edit the environment file with your credentials
nano .env
```
Fill in your specific node access tokens, proxy authentication values, and target endpoints within the `.env` file.

#### 3. Initialize Target Mount Vectors
Create the local state directories that the direct and heavy compute layers use for long-term persistence. This avoids container runtime file-locking issues:
```bash
mkdir -p titan_data bitping_data chromium_config
```

#### 4. Launch the Isolated Pipeline
Boot up the entire stack in detached background mode. Docker Compose will automatically build the network dependencies, spin up the `tun2socks` container first, and then attach the edge nodes inside its namespace:
```bash
docker compose up -d
```

#### 5. Verify Architecture Health
Ensure all network abstractions and resource restrictions are operational:
```bash
# View active container runtime limits and memory optimization
docker stats

# Verify routing and network stack integrity
docker compose ps
```

### Post-Deployment Security Auditing
To verify that the Network Namespace Isolation is working perfectly and that your edge nodes are hidden behind the proxy, run an interactive query against one of the isolated containers:
```bash
docker compose exec --index=1 tm_edge_node curl ifconfig.me
```
*Note: We use `tm_edge_node` here matching the container name defined in docker-compose.yml.*

**Expected Output**: The terminal should return your target Proxy IP, not your core VPS hosting provider's datacenter IP. If the proxy fails or drops, the curl command will timeout entirely rather than reverting to the host network—confirming the kill-switch works.

## Support & Custom Builds

Facing issues, or looking to build custom DevOps/Proxy infrastructure? My DMs are always open. Reach out on Telegram: @Jerrooooo


