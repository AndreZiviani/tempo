---
title: Deployment
weight: 250
---

# Deploying Tempo

Tempo can be easily deployed through a number of tools as explained in this document.
The linked folders contain example deployments of Tempo.  They are a good resource for getting some basic configurations together.

## Docker Compose

The [docker-compose](https://github.com/grafana/tempo/tree/main/example/docker-compose) examples are simpler and designed to show minimal configuration.  This is a great place
to get started with Tempo and learn about various trace discovery flows.

- [local storage](https://github.com/grafana/tempo/tree/main/example/docker-compose/readme.md#local-storage)
    - At its simplest Tempo only requires a few parameters that identify where to store traces.
- [s3/minio storage](https://github.com/grafana/tempo/tree/main/example/docker-compose/readme.md#s3)
    - To reduce complexity not all config options are exposed on the command line.  This example uses the minio/s3 backend with a config file.
- [Trace discovery with Loki](https://github.com/grafana/tempo/tree/main/example/docker-compose/readme.md#loki-derived-fields)
    - This example brings in Loki and shows how to use a log flow to discover traces.

## Jsonnet/Tanka

The [jsonnet](https://github.com/grafana/tempo/tree/main/example/tk) examples are more complex and show off the full range of configuration available to Tempo.  The
Helm and jsonnet examples are equivalent.  They are both provided for people who prefer different configuration
mechanisms.

- [single binary](https://github.com/grafana/tempo/tree/main/example/tk/readme.md#single-binary)
    - A single binary jsonnet deployment.  Valuable for getting started with advanced configuration.
- [microservices](https://github.com/grafana/tempo/tree/main/example/tk/readme.md#microservices)
    - Tempo as a set of independently scalable microservices.  This is recommended for high volume full production deployments.

## Helm

Helm charts are available in the grafana/helm-charts repo:

- [single binary](https://github.com/grafana/helm-charts/tree/main/charts/tempo)
- [microservices](https://github.com/grafana/helm-charts/tree/main/charts/tempo-distributed)

## The New Stack (TNS) Demo

The [TNS demo](https://github.com/grafana/tns) demonstrates a fully-instrumented three-tier application and the integration of Grafana, Prometheus, Loki, and Tempo [features](https://github.com/grafana/tns#demoable-things), including metrics to traces (exemplars), logs to traces, and traces to logs.  A good place to start is the [docker-compose setup](https://github.com/grafana/tns/tree/main/production/docker-compose) which includes a pre-built dashboard, load generator, and exemplars.

Explanation:
- Metrics To Traces (Exemplars)
    - The weaveworks middleware automatically [records](https://github.com/weaveworks/common/blob/bd288de53d57de300fa286688ce2fc935687213f/middleware/instrument.go#L79) request latency with an exemplar.  Try running the following PromQL query in Grafana `Explore` and enabling the exemplars switch. It shows the p50 request latency for the "app" container:  `histogram_quantile(0.5, sum(rate(tns_request_duration_seconds_bucket{job="tns/app"}[$__rate_interval])) by (le))`.  Click the exemplar to see the trace.
- LogqlV2 and Logs to Traces
    - The http client [logs inter-service http requests](https://github.com/grafana/tns/blob/main/client/http.go#L70) in `logfmt` format, which enables the ability to perform complex queries over api traffic. Try running the following query which shows all failed api requests from app to db and took longer than 100ms: `{job="tns/app"} | logfmt | level="info" and status>=500 and status <=599 and duration > 100ms`.  Expand the log line and click the Tempo button near the trace ID to see the trace.
- Traces To Logs
    - When viewing only a trace in the Explore view (i.e. not side-by-side with logs), the Logs icon will appear next to each span.  Click it to view the matching logs.
- Status
    - Exemplar support in Prometheus is still pre-release so a custom image is used, and the feature is enabled with the `--enable-feature=exemplar-storage` command line parameter.
