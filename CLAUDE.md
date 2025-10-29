# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is a Docker-based monitoring stack for tracking server resources, Docker container metrics, and PostgreSQL database metrics using Prometheus, Grafana, Node Exporter, cAdvisor, and Postgres Exporter. The system collects metrics every 15 seconds and retains 30 days of historical data.

## Architecture

**Components:**
- **Prometheus** (port 9090): Time-series database that scrapes and stores metrics
- **Node Exporter** (port 9100): Exposes host system metrics (CPU, RAM, disk, network)
- **cAdvisor** (port 8080): Exposes Docker container metrics
- **Postgres Exporter** (port 9187): Exposes PostgreSQL database metrics (connections, queries, performance)
- **Grafana** (port 3030): Visualization layer for dashboards (note: mapped to 3030 on host, 3000 in container)

**Key Design Decisions:**
- Node Exporter uses `network_mode: host` and `pid: host` to access host system information
- cAdvisor runs as privileged container with access to `/dev/kmsg` for kernel-level container metrics
- Postgres Exporter uses `host.docker.internal` to connect to PostgreSQL running on the host machine
- All services use `restart: always` for automatic recovery
- Data persists in named volumes: `prometheus-data` and `grafana-data`
- Services communicate via the `monitoring` bridge network
- PostgreSQL credentials are stored in `.env` file (not committed to git)

## Common Commands

### Starting/Stopping Services
```bash
# First time setup: Copy .env.example to .env and configure PostgreSQL credentials
cp .env.example .env
# Edit .env with your PostgreSQL connection details

# Start all services
docker compose up -d

# Stop all services
docker compose down

# Stop and remove volumes (deletes all metrics data)
docker compose down -v

# View running containers
docker compose ps
```

### Monitoring and Debugging
```bash
# View logs for all services
docker compose logs -f

# View logs for specific service
docker compose logs -f prometheus
docker compose logs -f grafana
docker compose logs -f node-exporter
docker compose logs -f cadvisor
docker compose logs -f postgres-exporter

# Check Prometheus targets health
curl http://localhost:9090/api/v1/targets | grep -o '"health":"[^"]*"'

# Test node-exporter metrics
curl http://localhost:9100/metrics | grep node_cpu

# Test cAdvisor metrics
curl http://localhost:8080/metrics | grep container_cpu

# Test postgres-exporter metrics
curl http://localhost:9187/metrics | grep pg_

# Restart specific service
docker compose restart prometheus
docker compose restart grafana
docker compose restart postgres-exporter
```

### Configuration Changes
```bash
# After modifying prometheus.yml, reload Prometheus without restart
docker compose exec prometheus kill -HUP 1

# Or restart Prometheus to apply changes
docker compose restart prometheus
```

## Configuration Files

### prometheus.yml
- `scrape_interval: 15s` - Controls metric collection frequency
- `storage.tsdb.retention.time=30d` - Data retention period (set in docker-compose.yml)
- Four scrape jobs: prometheus (self-monitoring), node-exporter (host metrics), cadvisor (container metrics), postgres-exporter (PostgreSQL metrics)
- All targets use service names for DNS resolution within Docker network

### docker-compose.yml
- Grafana uses non-standard host port 3030 (instead of 3000) to avoid conflicts
- Node Exporter filesystem collectors are configured to exclude virtual/system filesystems
- Default Grafana credentials: admin/admin (should be changed on first login)
- Postgres Exporter requires `.env` file with PostgreSQL connection details

### .env
- Required for Postgres Exporter to connect to PostgreSQL
- Contains `POSTGRES_USER`, `POSTGRES_PASSWORD`, `POSTGRES_DB`, `POSTGRES_PORT`
- Copy from `.env.example` and fill in actual values
- Not committed to git (sensitive credentials)

## Accessing Services

- Grafana UI: `http://localhost:3030` (default: admin/admin)
- Prometheus UI: `http://localhost:9090`
- cAdvisor UI: `http://localhost:8080`
- Node Exporter metrics: `http://localhost:9100/metrics`
- Postgres Exporter metrics: `http://localhost:9187/metrics`

## Recommended Grafana Dashboards

Import these dashboard IDs in Grafana:
- Dashboard 1860: Node Exporter Full (comprehensive host metrics)
- Dashboard 193: Docker Container Metrics
- Dashboard 9628: PostgreSQL Database (includes query performance, connections, cache hit ratio)
- Dashboard 11600: Combined Docker Monitoring (optional)

When importing, select the Prometheus data source configured at `http://prometheus:9090`.
