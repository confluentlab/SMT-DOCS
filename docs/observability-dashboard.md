# Observability Dashboard

The SMT registers two JMX MBeans per connector. Each connector gets its own instance, so you won't get collisions when running multiple connectors in the same Connect worker.

## JMX object names

```
connect.smt:type=DynamicJsonExtract,connector="<connector-name>"
connect.smt:type=SmtHealth,connector="<connector-name>"
```

## What to put on your dashboard

### Transform metrics

- **RecordsProcessed** — total successful transforms (use as a rate)
- **RecordsFailed** — total failures (rate this too)
- **RecordsSkipped** — records that had no mapping match
- **AvgTransformLatencyMs** — running average
- **MaxTransformLatencyMs** — worst case, useful for spotting outliers
- **MappingLookupMisses** — bumps when a record's connector/topic has no mapping entry
- **HotReloadSuccessCount / HotReloadFailureCount** — track live config updates
- **Transform cache hit ratio (derived)** — compute from cache hits and misses; should generally stay high

### Health status

- **Status** — `HEALTHY`, `DEGRADED`, or `UNHEALTHY`
- **ErrorRate5Min** — error rate over a sliding 5-minute window
- **LastReloadStatus** — `SUCCESS`, `FAILED: <reason>`, or `N/A`
- **LastReloadTime** — ISO timestamp of the last config reload
- **UptimeSeconds** — time since the SMT was configured

## Alert thresholds

These are starting points. Tune them based on your traffic patterns.

### Warning

- Error rate above 10% for 5 minutes: `ErrorRate5Min > 0.10`
- Avg latency above 50ms for 10 minutes: `AvgTransformLatencyMs > 50`
- Hot-reload failure count goes up in the last 15 minutes

### Critical

- Error rate above 50% for 2 minutes: `ErrorRate5Min > 0.50`
- Failure rate exceeds success rate for 3 minutes
- Health status is `UNHEALTHY`

### Worth investigating

- Sudden jump in `MappingLookupMisses` — usually means a new topic showed up that's not in the mapping file
- Cache hit ratio drops below 0.70 — might indicate unusual path variety or cache eviction pressure

## Prometheus JMX exporter

If you're using the JMX exporter sidecar with Connect, use these patterns:

```yaml
rules:
  - pattern: 'connect.smt<type=DynamicJsonExtract,connector=(.+)><>(\\w+)'
    name: smt_$2
    labels:
      connector: "$1"
    type: GAUGE
  - pattern: 'connect.smt<type=SmtHealth,connector=(.+)><>(\\w+)'
    name: smt_health_$2
    labels:
      connector: "$1"
    type: GAUGE
```

This gives you metrics like `smt_RecordsProcessed{connector="my-sink"}` in Prometheus. From there, set up Grafana panels however you like.
