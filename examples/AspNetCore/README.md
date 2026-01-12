# OpenTelemetry ASP.NET Core Web API Example

This example uses the new WebApplication host that ships with .NET
and shows how to setup

1. OpenTelemetry logging
2. OpenTelemetry metrics
3. OpenTelemetry tracing

`ResourceBuilder` is associated with OpenTelemetry to associate the
service name, version and the machine on which this program is running.

The sample rate is set to emit all the traces using `AlwaysOnSampler`.
You can try out different samplers like `TraceIdRatioBasedSampler`.

## Architecture

```mermaid
flowchart LR
    subgraph App
        AspNetCore["ASP.NET Core App"]
    end

    subgraph Collector["OTel Collector (OTLP/gRPC :4317, OTLP/HTTP :4318)"]
        OTLP_Receiver["OTLP Receiver<br/>(gRPC :4317, HTTP :4318)"]
        Traces_Pipeline["Traces Pipeline"]
        Metrics_Pipeline["Metrics Pipeline<br/>(Prometheus exporter :9201)"]
        Logs_Pipeline["Logs Pipeline"]
    end

    subgraph Backends
        Tempo["Tempo (OTLP/gRPC :4317, OTLP/HTTP :4318; API :3200)"]
        Prometheus["Prometheus :9090"]
    end

    subgraph UI
        Grafana["Grafana :3000"]
    end

    AspNetCore -->|"OTLP (gRPC :4317 / HTTP :4318)"| OTLP_Receiver
    OTLP_Receiver --> Traces_Pipeline
    OTLP_Receiver --> Metrics_Pipeline
    OTLP_Receiver --> Logs_Pipeline

    Traces_Pipeline -->|"OTLP/gRPC :4317"| Tempo
    Prometheus -->|"scrape http :9201"| Metrics_Pipeline

    Tempo --> Grafana
    Prometheus --> Grafana
```

## Running Dependencies via Docker

The example by default writes telemetry to stdout. To enable telemetry export
via OTLP, update the `appsettings.json` file to replace `"console"` with
`"otlp"`. Launching the application will then send telemetry data via OTLP.

Use the provided "docker-compose.yaml" file to spin up the
required dependencies, including:

- **OTel Collector** Accept telemetry and forwards them to Tempo, Prometheus
- **Prometheus** to store metrics
- **Grafana (UI)** UI to view metrics, traces. (Exemplars can be used to jump
  from metrics to traces)
- **Tempo** to store traces // TODO: Add a logging store also.

Once the Docker containers are running, you can access the **Grafana UI** at:
[http://localhost:3000/](http://localhost:3000/)

## References

- [ASP.NET Core](https://learn.microsoft.com/aspnet/core/introduction-to-aspnet-core)
- [Docker](http://docker.com)
- [Prometheus](http://prometheus.io/docs)
- [Tempo](https://github.com/grafana/tempo)
