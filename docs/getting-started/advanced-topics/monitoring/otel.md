---
sidebar_position: 7
title: "🔭 OpenTelemetry"
---

Open WebUI supports **distributed tracing and metrics** export via the OpenTelemetry (OTel) protocol (OTLP). This enables integration with monitoring systems like Grafana LGTM Stack, Jaeger, Tempo, and Prometheus to monitor request flows, database/Redis queries, response times, and more in real-time.

## 🚀 Quick Start with Docker Compose

The fastest way to get started with observability is using the pre-configured Docker Compose setup:

```bash
# Minimal setup: WebUI + Grafana LGTM all-in-one
docker compose -f docker-compose.otel.yaml up -d
```

The `docker-compose.otel.yaml` file starts the following services:

| Service | Port | Description |
|---------|------|-------------|
| **grafana/otel-lgtm** | 3000 (UI), 4317 (OTLP gRPC), 4318 (OTLP HTTP) | Loki + Grafana + Tempo + Mimir all-in-one |
| **open-webui** | 8080 → 8080 | WebUI with OTEL environment variables configured |

After startup, visit `http://localhost:3000` and log in with `admin` / `admin` to access the Grafana dashboard.

## ⚙️ Environment Variables

Configure OpenTelemetry using these environment variables:

| Variable | Default | Description |
|----------|---------|-------------|
| `ENABLE_OTEL` | **false** | Set to `true` to enable trace export |
| `ENABLE_OTEL_METRICS` | **false** | Enable FastAPI HTTP metrics export |
| `OTEL_EXPORTER_OTLP_ENDPOINT` | `http://localhost:4317` | OTLP gRPC/HTTP Collector URL |
| `OTEL_EXPORTER_OTLP_INSECURE` | `true` | Disable TLS (for local testing) |
| `OTEL_SERVICE_NAME` | `open-webui` | Service name tag in resource attributes |
| `OTEL_BASIC_AUTH_USERNAME` / `OTEL_BASIC_AUTH_PASSWORD` | _(empty)_ | Basic Auth credentials for Collector if required |

> You can override these in your `.env` file or `docker-compose.*.yaml` configuration.

## 📊 Data Collection

### Distributed Tracing

The `utils/telemetry/instrumentors.py` module automatically instruments the following libraries:

* **FastAPI** (request routes) · **SQLAlchemy** · **Redis** · External calls via `requests` / `httpx` / `aiohttp`
* Span attributes include:
  * `db.instance`, `db.statement`, `redis.args`
  * `http.url`, `http.method`, `http.status_code`
  * Error details: `error.message`, `error.kind` when exceptions occur


### Metrics Collection

The `utils/telemetry/metrics.py` module exports the following metrics:

| Instrument | Type | Unit | Labels |
|------------|------|------|--------|
| `http.server.requests` | Counter | 1 | `http.method`, `http.route`, `http.status_code` |
| `http.server.duration` | Histogram | ms | Same as above |

Metrics are pushed to the Collector (OTLP) every 10 seconds and can be visualized in Prometheus → Grafana.

## 🔧 Custom Collector Setup

If you're running your own OpenTelemetry Collector:

```bash
# Start Open WebUI
docker run -d --name open-webui \
  -p 8080:8080 \
  -e ENABLE_OTEL=true \
  -e OTEL_EXPORTER_OTLP_ENDPOINT=http://your-collector:4317 \
  -v open-webui:/app/backend/data \
  ghcr.io/open-webui/open-webui:main
```

## 🚨 Troubleshooting

### Common Issues

**Traces not appearing in Grafana:**
- Verify `ENABLE_OTEL=true` is set
- Check collector connectivity: `curl http://localhost:4317`
- Review Open WebUI logs for OTLP export errors

**Authentication issues:**
- Set `OTEL_BASIC_AUTH_USERNAME` and `OTEL_BASIC_AUTH_PASSWORD` for authenticated collectors
- Verify TLS settings with `OTEL_EXPORTER_OTLP_INSECURE`