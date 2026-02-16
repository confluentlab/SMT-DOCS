# OpenTelemetry Integration

The SMT can emit transform spans when OpenTelemetry API jars are on your Connect plugin classpath. It's completely optional — if the OTel classes aren't there, nothing breaks. The SMT logs a message once and moves on.

## Config

| Property | Default | What it does |
|----------|---------|-------------|
| `app.otel.enabled` | `false` | Turn on span creation |
| `app.otel.detailed` | `false` | Emit per-field `transform.step` events inside the span |
| `app.otel.instrumentationName` | `connect-smt` | The instrumentation scope name that shows up in your traces |

## How it works

The SMT uses reflection to find OpenTelemetry's `GlobalOpenTelemetry.getTracer()`. That way there's no compile-time dependency on OTel jars — you can ship the same SMT JAR to clusters that have OTel and clusters that don't.

Each `apply()` call creates a span called `smt.transform` with these attributes:

- `connector` — connector name from config
- `topic` — Kafka topic
- `partition` — Kafka partition number

When `app.otel.detailed=true`, the SMT also emits an event for every field transform step (toString, dateFormat, mask, encrypt). Each event includes the field name, transform type, and before/after values. This is useful for debugging but adds overhead at high throughput, so leave it off unless you're investigating something specific.

## What to keep in mind

- This is best-effort. If span creation fails for any reason, the transform still runs normally. Records never fail because of tracing.
- The SMT doesn't bundle OpenTelemetry JARs. You need to provide `opentelemetry-api` on the classpath yourself, usually through the Java agent or Connect plugin dependencies.
- The detailed mode (`app.otel.detailed=true`) generates one event per field per record. At 50K records/sec with 10 fields each, that's 500K events/sec. Only turn it on when you need it.
