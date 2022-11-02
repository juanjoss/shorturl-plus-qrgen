# shorturl-plus-qrgen

Configuration repository to run [qrgen](https://github.com/juanjoss/qrgen) and [shorturl](https://github.com/juanjoss/shorturl) with Docker Compose.

## Run it

```bash
docker compose up -d
```

## Configuration

The configuration files for the otel collector, jaeger and prometheus instances are located at [/config](https://github.com/juanjoss/shorturl-plus-qrgen/tree/master/config).

### qrgen

Runs the metrics server at port `5000` and the gRPC server at port `5001`. 

Environment configuration is:

- HTTP_METRICS_PORT (HTTP server port to expose metrics at `/metrics`)
- GRPC_SERVER_PORT (grpc server port)
- GRPC_OTEL_COLLECTOR (otel collector OTLP gRPC receiver)

### shorturl

Runs the HTTP server at port `4000`.

Environment configuration is:

- APP_PORT (HTTP server port)
- REDIS_PORT (redis instance port)
- QRGEN_GRPC_SERVER_PORT (qrgen gRPC server port)
- GRPC_OTEL_COLLECTOR (otel collector OTLP gRPC receiver)

Endpoints:

- GET / (main page)
- POST /url (shorten URL)
- GET /{id} (resolve to original URL and redirect)
- GET /qr/{id} (get QR code)
- GET /api/cache (get all content from cache)
- GET /api/metrics (metrics)

## Instrumentation

Both services are instrumented using the [OpenTelemtry Go SDK](https://opentelemetry.io/docs/instrumentation/go/).

### Tracing

Traces are implemented using the Go SDK. When produced they're sent to the open telemetry collector instance through a gRPC endpoint using OTLP (open telemetry protocol), batch processed, exported and loaded to a Jaeger instance (write endpoint port is `14250`). The Jaeger UI runs at port `16686`.

### Metrics

Metrics are exposed through a HTTP endpoint and consumed by the Prometheus instance. The endpoints are `/metrics` for qrgen and `/api/metrics` for shorturl.

_Metrics could be instrumented manually with the open telemetry SDK and sent to the otel collector to be exported to a external prometheus instance, but [the Go SDK does not support this yet](https://opentelemetry.io/docs/instrumentation/go/manual/#creating-metrics)._

### Logs

Logs are generated using the default gorilla/mux logging handler, but they're not collected in any way.

## Diagram

![arch_basic](https://drive.google.com/uc?export=view&id=1V6u29xwERYyenkXIaoNaCBrQvdxlne-x)
