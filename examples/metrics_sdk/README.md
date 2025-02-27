# OpenTelemetry Ruby Metrics SDK Example

## metrics_collect.rb

Run the script to see the metric data from console

```sh
ruby metrics_collect.rb
```

## metrics_collect_otlp.rb

**WARN: this example doesn't work on alpine aarch64 container due to grpc installation issues.**

This example tests both the metrics sdk and the metrics otlp http exporter.

You can view the metrics in your favored backend (e.g. jaeger).

### 1. Set up the local opentelemetry-collector.

Given you have a `config.yml` file in your current directory and Docker is installed on your machine, run the following commands to pull the collector image and run the collector.

```sh
docker pull otel/opentelemetry-collector
docker run --rm -v $(pwd)/config.yaml:/etc/otel/config.yaml -p 4317:4317 -p 4318:4318 otel/opentelemetry-collector --config /etc/otel/config.yaml
```

Sample config.yaml

```yaml
receivers:
  otlp:
    protocols:
      grpc:
        endpoint: 0.0.0.0:4317
      http:
        endpoint: 0.0.0.0:4318

exporters:
  debug:
    verbosity: detailed

processors:
  batch:

service:
  pipelines:
    traces:
      receivers: [otlp]
      processors: [batch]
      exporters: [debug]
    metrics:
      receivers: [otlp]
      processors: [batch]
      exporters: [debug]
```

More information on how to setup the OTel collector can be found in the in [quick start docs](https://opentelemetry.io/docs/collector/quick-start/).

### 2. Assign the endpoint value to your destination address

```sh
# Using environment variable
ENV['OTEL_EXPORTER_OTLP_METRICS_ENDPOINT'] = 'http://host.docker.internal:4318/v1/metrics'

# Or using export command
export OTEL_EXPORTER_OTLP_METRICS_ENDPOINT=http://host.docker.internal:4318/v1/metrics
```

### 3. Run the script to send metric data to OTLP collector

```sh
ruby metrics_collect_otlp.rb
```

You should see metric data appear in the collector.
