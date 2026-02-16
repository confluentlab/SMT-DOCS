# Kafka Connect SMT — Dynamic JSON Extract

A single-message transform for Kafka Connect that takes raw JSON off your topics and reshapes it into whatever your sink needs. You write a mapping file, point it at a connector, and the SMT handles extraction, date formatting, masking, encryption, and database-specific output formatting.

## Quick start

Build the shaded JAR and drop it into your Connect plugin directory:

```bash
mvn clean package
# output: target/connect-smt-1.0.0.jar
```

Then add it to a connector config:

```properties
name=mongo-assessment-sink
connector.class=com.mongodb.kafka.connect.MongoSinkConnector
topics=fhir-assessments

transforms=Extract
transforms.Extract.type=com.example.fhir.connect.smt.DynamicJsonExtractSMT
transforms.Extract.connector.name=mongo-assessment-sink
```

That's it. The SMT looks up the mapping for `mongo-assessment-sink`, extracts the fields you care about, and outputs a document that MongoDB knows how to write.

## How it works

1. Reads the record value as bytes (handles `byte[]`, `String`, `ByteBuffer`, `Map`, `Struct`)
2. Parses JSON using Jackson
3. Looks up the mapping by connector name (falls back to topic name if no connector mapping exists)
4. Walks the output tree, extracting fields via JSONPath-like expressions
5. Applies any transforms you've configured (date formatting, masking, encryption, toString)
6. Formats the result for your target database

## Target databases

Set `target.database` in your connector config. The SMT supports a core set of native database targets and can be used with additional systems through connector-level compatibility patterns. See [supported-databases.md](docs/supported-databases.md) for guidance.

| Type | Config value | Output format |
|------|-------------|---------------|
| MongoDB | `mongodb` | Schemaless map |
| MySQL | `mysql` | Schema + Struct |
| PostgreSQL | `postgresql` | Schema + Struct |
| SQL Server | `sqlserver` | Schema + Struct |
| Oracle | `oracle` | Schema + Struct |
| Snowflake | `snowflake` | JSON string payload |
| Delta Lake | `delta_lake` | Schema + Struct |
| Embedded (SQLite, DuckDB) | `embedded` | Schema + Struct |
| Neo4j | `neo4j` | Schemaless map |
| InfluxDB | `influxdb` | Schemaless map |
| Vector DB | `vectordb` | Schemaless map |
| RDF | `rdf` | Schemaless map |
| Ledger | `ledger` | Schemaless map |
| Object Store | `object_store` | Schemaless map |

## Connector config

Config values are set as `transforms.<name>.<key>` in your connector properties file.

| Property | Default | What it does |
|----------|---------|-------------|
| `connector.name` | — | Name used to look up mappings. Should match the mapping file key |
| `target.database` | `mongodb` | Which formatter to use |
| `mapping.classpathRoot` | `mappings/topic-mappings.json` | Path to the mapping file inside the JAR |
| `mapping.source` | `auto` | Where to load mappings from: `classpath`, `s3`, or `auto` (tries S3 first, falls back to classpath) |
| `mapping.s3.endpoint` | — | S3/MinIO endpoint URL |
| `mapping.s3.bucket` | — | S3 bucket name |
| `mapping.s3.key` | — | Object key for the mapping file |
| `mapping.s3.accessKey` | — | S3 access key (stored as PASSWORD type) |
| `mapping.s3.secretKey` | — | S3 secret key (stored as PASSWORD type) |
| `mapping.s3.region` | `us-east-1` | S3 region |
| `mapping.s3.maxAttempts` | `3` | Retry attempts for S3 operations |
| `mapping.s3.initialBackoffMs` | `200` | Initial backoff for S3 retries |
| `mapping.hotreload.enabled` | `false` | Poll S3 for mapping changes |
| `mapping.hotreload.intervalSeconds` | `30` | How often to check for changes |
| `app.failOnMissingMapping` | `false` | Log error instead of warn when no mapping matches |
| `app.attachKafkaMetadata` | `true` | Add `_kafka: {topic, partition}` to the output |
| `app.storeRawPayload` | `false` | Include the original JSON as `_raw` |
| `app.trace.enabled` | `false` | Log per-record transform details at TRACE level |
| `app.metrics.enabled` | `true` | Register JMX metrics |
| `app.otel.enabled` | `false` | Emit OpenTelemetry spans |
| `app.otel.detailed` | `false` | Add per-field transform events to OTel spans |
| `app.otel.instrumentationName` | `connect-smt` | OTel instrumentation scope name |
| `jdbc.flatten.nested` | `true` | Flatten nested JSON objects into columns |
| `jdbc.array.mode` | `json_string` | How to handle arrays: `json_string`, `first_only`, `native_array`, `explode` |
| `jdbc.column.separator` | `_` | Separator for flattened column names |

## Config precedence

The SMT resolves config in this order:

1. Connector config (`transforms.<name>.*`)
2. `smt-defaults.properties` (shipped inside the JAR)
3. Built-in fallback defaults

Some mapping-related keys also support environment-variable fallbacks.

## Connector examples

### PostgreSQL JDBC sink

```properties
name=postgres-claims-sink
connector.class=io.confluent.connect.jdbc.JdbcSinkConnector
topics=claims-events

transforms=Extract
transforms.Extract.type=com.example.fhir.connect.smt.DynamicJsonExtractSMT
transforms.Extract.connector.name=postgres-claims-sink
transforms.Extract.target.database=postgresql
transforms.Extract.jdbc.flatten.nested=true
transforms.Extract.jdbc.array.mode=native_array
```

### Snowflake sink

```properties
name=snowflake-events-sink
connector.class=com.snowflake.kafka.connector.SnowflakeSinkConnector
topics=events

transforms=Extract
transforms.Extract.type=com.example.fhir.connect.smt.DynamicJsonExtractSMT
transforms.Extract.connector.name=snowflake-events-sink
transforms.Extract.target.database=snowflake
```

### S3-sourced mappings with hot-reload

```properties
transforms.Extract.type=com.example.fhir.connect.smt.DynamicJsonExtractSMT
transforms.Extract.connector.name=my-sink
transforms.Extract.mapping.source=auto
transforms.Extract.mapping.s3.endpoint=http://minio:9000
transforms.Extract.mapping.s3.bucket=config
transforms.Extract.mapping.s3.key=mappings/topic-mappings.json
transforms.Extract.mapping.s3.accessKey=${S3_ACCESS_KEY}
transforms.Extract.mapping.s3.secretKey=${S3_SECRET_KEY}
transforms.Extract.mapping.hotreload.enabled=true
transforms.Extract.mapping.hotreload.intervalSeconds=30
```

## Observability

The SMT exposes two JMX MBeans per connector when `app.metrics.enabled=true`:

**Transform metrics** — `connect.smt:type=DynamicJsonExtract,connector="<name>"`
Tracks records processed/failed/skipped, latency, payload bytes, hot-reload counts, encrypt/mask calls, and cache stats.

**Health status** — `connect.smt:type=SmtHealth,connector="<name>"`
Reports HEALTHY / DEGRADED / UNHEALTHY based on 5-minute error rate. Supports `forceReload()` via JMX.

See [observability-dashboard.md](docs/observability-dashboard.md) for dashboard setup, alert thresholds, and Prometheus exporter config.

OpenTelemetry is opt-in — set `app.otel.enabled=true` and the SMT will emit spans using reflection (no hard dependency). See [open-telemetry.md](docs/open-telemetry.md).

Structured logging uses MDC keys (`connector`, `topic`, `partition`) so you can filter logs per connector. Enable per-record tracing with `app.trace.enabled=true`.

## Benchmarks

Run microbenchmarks with the `jmh` profile:

```bash
mvn -Pjmh -DskipTests compile exec:java \
  -Dexec.mainClass=org.openjdk.jmh.Main \
  -Dexec.args="com.example.fhir.connect.smt.benchmark.TransformEngineBenchmark -wi 3 -i 5 -f 0"
```

## Docs

| Doc | What's in it |
|-----|-------------|
| [supported-databases.md](docs/supported-databases.md) | Full database compatibility list |
| [smt-workflow.md](docs/smt-workflow.md) | Internal flow diagram |
| [observability-dashboard.md](docs/observability-dashboard.md) | JMX dashboards and alerting |
| [open-telemetry.md](docs/open-telemetry.md) | OpenTelemetry integration |
| [release-versioning.md](docs/release-versioning.md) | Versioning and release process |
| [setup-runner.md](docs/setup-runner.md) | GitHub Actions self-hosted runner |
