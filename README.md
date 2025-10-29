# Server & Docker & PostgreSQL Monitoring with Prometheus + Grafana

A complete monitoring solution for tracking server resources (CPU, RAM, disk, network), Docker container metrics, and PostgreSQL database performance in real-time with historical data.

## 📋 Table of Contents

- [Features](#features)
- [Architecture](#architecture)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Usage](#usage)
- [Troubleshooting](#troubleshooting)
- [Maintenance](#maintenance)
- [Advanced Configuration](#advanced-configuration)

## ✨ Features

### Host Server Monitoring
- ✅ Real-time CPU usage (total and per core)
- ✅ Memory usage and availability
- ✅ Disk space and I/O operations
- ✅ Network traffic (bytes in/out)
- ✅ System load average
- ✅ Uptime tracking

### Docker Container Monitoring
- ✅ CPU usage per container
- ✅ Memory usage per container
- ✅ Network I/O per container
- ✅ Disk I/O per container
- ✅ Container status and restart counts
- ✅ All metrics from `docker stats`

### PostgreSQL Database Monitoring
- ✅ Active connections and connection limits
- ✅ Query execution times and performance
- ✅ Database size and table statistics
- ✅ Cache hit ratios
- ✅ Transaction rates (commits/rollbacks)
- ✅ Locks and deadlocks
- ✅ Replication lag (if applicable)

### Data & Visualization
- ✅ 30 days of historical data retention (configurable)
- ✅ 15-second metric collection interval
- ✅ Beautiful, customizable Grafana dashboards
- ✅ Real-time updates
- ✅ Zoom and drill-down capabilities

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────┐
│                     Your Server                         │
│                                                          │
│  ┌──────────────┐  ┌─────────────┐  ┌──────────────┐  │
│  │ Node Exporter│  │  cAdvisor   │  │   Postgres   │  │
│  │ (Port 9100)  │  │ (Port 8080) │  │   Exporter   │  │
│  │              │  │             │  │ (Port 9187)  │  │
│  │ Host Metrics │  │  Container  │  │   Database   │  │
│  │ CPU/RAM/Disk │  │   Metrics   │  │   Metrics    │  │
│  └──────┬───────┘  └──────┬──────┘  └──────┬───────┘  │
│         │                  │                 │          │
│         └──────────────────┼─────────────────┘          │
│                            ▼                            │
│                  ┌─────────────────┐                    │
│                  │   Prometheus    │                    │
│                  │   (Port 9090)   │                    │
│                  │                 │                    │
│                  │ Time-Series DB  │                    │
│                  └────────┬────────┘                    │
│                           ▼                             │
│                  ┌─────────────────┐                    │
│                  │    Grafana      │                    │
│                  │   (Port 3030)   │◄─── User Access   │
│                  │                 │                    │
│                  │  Visualization  │                    │
│                  └─────────────────┘                    │
│                                                          │
│  ┌──────────────────────────────────────────────────┐  │
│  │         PostgreSQL (Host/External)               │  │
│  │         Monitored via Postgres Exporter          │  │
│  └──────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────┘
```

## 📦 Components

| Component | Purpose | Port |
|-----------|---------|------|
| **Prometheus** | Collects and stores metrics | 9090 |
| **Node Exporter** | Exposes host system metrics | 9100 |
| **cAdvisor** | Exposes Docker container metrics | 8080 |
| **Postgres Exporter** | Exposes PostgreSQL database metrics | 9187 |
| **Grafana** | Visualizes metrics with dashboards | 3030 |

## 🔧 Prerequisites

- Docker installed (version 20.10+)
- Docker Compose installed (version 2.0+)
- Linux-based server (Ubuntu, Debian, CentOS, etc.)
- Minimum 2GB RAM available
- Root or sudo access
- PostgreSQL installed (optional, for database monitoring)

## 🚀 Installation

### Step 1: Configure PostgreSQL Connection (Optional)

If you want to monitor PostgreSQL, create a `.env` file:

```bash
# Copy the example file
cp .env.example .env

# Edit with your PostgreSQL credentials
nano .env
```

Example `.env` content:
```env
POSTGRES_USER=postgres
POSTGRES_PASSWORD=your_password
POSTGRES_DB=postgres
POSTGRES_PORT=5432
```

**Optional: Create a dedicated monitoring user in PostgreSQL:**
```sql
CREATE USER postgres_exporter WITH PASSWORD 'your_password';
GRANT pg_monitor TO postgres_exporter;
```

If you don't need PostgreSQL monitoring, you can skip this step and the postgres-exporter service will fail gracefully.

### Step 2: Start the Monitoring Stack

```bash
# Start all services
docker compose up -d

# Verify all containers are running
docker compose ps

# Expected output: 5 containers in "Up" status (or 4 if skipping PostgreSQL)
```

### Step 3: Verify Services

```bash
# Check Prometheus targets
curl http://localhost:9090/api/v1/targets | grep -o '"health":"[^"]*"'

# Check node-exporter metrics
curl http://localhost:9100/metrics | grep node_cpu

# Check cAdvisor metrics
curl http://localhost:8080/metrics | grep container_cpu

# Check postgres-exporter metrics (if configured)
curl http://localhost:9187/metrics | grep pg_

# View logs
docker compose logs -f

# View specific service logs
docker compose logs -f postgres-exporter
```

## ⚙️ Configuration

### Grafana Setup

1. **Access Grafana**
   ```
   http://your-server-ip:3000
   ```

2. **Login**
   - Username: `admin`
   - Password: `admin`
   - Change password when prompted

3. **Add Prometheus Data Source**
   - Click **Add data source**
   - Select **Prometheus**
   - URL: `http://prometheus:9090`
   - Click **Save & Test**
   - Should show ✅ "Data source is working"

4. **Import Dashboards**

   **Dashboard 1: Node Exporter Full (Host Metrics)**
   - Click **+** → **Import**
   - Enter ID: `1860`
   - Click **Load**
   - Select Prometheus data source
   - Click **Import**

   **Dashboard 2: Docker Container Metrics**
   - Click **+** → **Import**
   - Enter ID: `193`
   - Click **Load**
   - Select Prometheus data source
   - Click **Import**

   **Dashboard 3: PostgreSQL Database (If Monitoring PostgreSQL)**
   - Click **+** → **Import**
   - Enter ID: `9628`
   - Click **Load**
   - Select Prometheus data source
   - Click **Import**

   **Dashboard 4: Combined Docker Monitoring (Optional)**
   - Click **+** → **Import**
   - Enter ID: `11600`
   - Click **Load**
   - Select Prometheus data source
   - Click **Import**

## 🔍 Usage

| Service | URL | Purpose |
|---------|-----|---------|
| Grafana | http://your-server-ip:3030 | View dashboards |
| Prometheus | http://your-server-ip:9090 | Query metrics, check targets |
| cAdvisor | http://your-server-ip:8080 | View raw container metrics |
| Node Exporter | http://your-server-ip:9100/metrics | View raw host metrics |
| Postgres Exporter | http://your-server-ip:9187/metrics | View raw PostgreSQL metrics |

## 🔗 Resources

- [Prometheus Documentation](https://prometheus.io/docs/)
- [Grafana Documentation](https://grafana.com/docs/)
- [Node Exporter](https://github.com/prometheus/node_exporter)
- [cAdvisor](https://github.com/google/cadvisor)
- [Grafana Dashboards](https://grafana.com/grafana/dashboards/)
