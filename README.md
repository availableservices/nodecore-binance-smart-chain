# BSC RPC Node (DRPC Nodecore)

A lightweight RPC proxy for **BNB Smart Chain (BSC)** powered by [DRPC Nodecore](https://github.com/drpcorg/nodecore). Routes JSON-RPC requests to BSC public endpoints with intelligent load balancing, caching, and failover.

[![Powered by dRPC](https://drpc.org/images/external/powered-by-drpc-dark.svg)](https://drpc.org?ref=778cf4)

## Overview

| | |
|---|---|
| **Chain** | BNB Smart Chain (BSC) |
| **Upstream Provider** | `https://bsc.drpc.org` |
| **RPC Port** | `9090` |
| **Metrics Port** | `9093` |
| **RPC Endpoint** | `http://localhost:9090/queries/bsc` |

## Prerequisites

- [Docker](https://docs.docker.com/get-docker/) and [Docker Compose](https://docs.docker.com/compose/install/) installed

## Project Structure

```
rpc-custom/
├── docker-compose-nodecore.yml   # Docker Compose configuration
├── config/
│   └── nodecore.yml              # Nodecore upstream configuration
└── README.md
```

## Setup

### 1. Clone the repository

```bash
git clone <repo-url>
cd rpc-custom
```

### 2. (Optional) Configure upstream

Edit `config/nodecore.yml` to change the upstream RPC provider or add multiple upstreams:

```yaml
server:
  port: 9090
  metrics-port: 9093

upstream-config:
  upstreams:
    - id: drpc-bsc
      chain: bsc
      connectors:
        - type: json-rpc
          url: https://bsc.drpc.org
```

> To add more upstream providers, duplicate the upstream block with a different `id` and `url`. Nodecore will automatically load-balance between them.

### 3. Start the node

```bash
docker compose -f docker-compose-nodecore.yml up -d
```

### 4. Verify it's running

```bash
docker logs nodecore
```

You should see:

```
starting an rpc head of upstream drpc-bsc with poll interval 1m0s
http server started on [::]:9090
http server started on [::]:9093
```

> **Note:** Wait ~30–60 seconds for the upstream to sync the latest block head before sending requests.

## Usage

### JSON-RPC endpoint

```
POST http://localhost:9090/queries/bsc
Content-Type: application/json
```

### Example: Get latest block number

```bash
curl -X POST http://localhost:9090/queries/bsc \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}'
```

Response:

```json
{"id":1,"jsonrpc":"2.0","result":"0x585d198"}
```

### Example: Get latest block

```bash
curl -X POST http://localhost:9090/queries/bsc \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"eth_getBlockByNumber","params":["latest",false],"id":1}'
```

### WebSocket endpoint

```
ws://localhost:9090/queries/bsc
```

### Metrics endpoint

Prometheus-compatible metrics are available at:

```
http://localhost:9093
```

## Management

### Stop the node

```bash
docker compose -f docker-compose-nodecore.yml down
```

### Restart the node

```bash
docker compose -f docker-compose-nodecore.yml restart
```

### View logs

```bash
docker logs -f nodecore
```

## Configuration Reference

### `config/nodecore.yml`

| Field | Description |
|---|---|
| `server.port` | HTTP/WebSocket RPC listening port |
| `server.metrics-port` | Prometheus metrics port |
| `upstream-config.upstreams[].id` | Unique identifier for the upstream |
| `upstream-config.upstreams[].chain` | Chain identifier (e.g. `bsc`, `ethereum`) |
| `upstream-config.upstreams[].connectors[].url` | Upstream RPC URL |

Full configuration documentation: [DRPC Nodecore Docs](https://github.com/drpcorg/nodecore/blob/main/docs/nodecore)

[![Powered by dRPC](https://drpc.org/images/external/powered-by-drpc-dark.svg)](https://drpc.org?ref=778cf4)

## License

This configuration is provided as-is. Nodecore is licensed under the [MIT License](https://github.com/drpcorg/nodecore/blob/main/LICENSE).
