# OpenTelemerty docker
otel collector, loki, grafana, prometheus, tempo

## Deploy
Clone repo
```
mkdir grafana-data loki-data prometheus-data tempo-data
chmod 777 grafana-data loki-data prometheus-data tempo-data
docker-compose up -d
```

## Endpoints
* Grafana: http://{host}:3000 - web UI
* Otel Collector: http://{host}:4318 - OLTP HTTP for metrics, logs and traces

## How to run the Java app
Download opentelemetry-javaagent.jar and put it somewhere. 

(https://github.com/open-telemetry/opentelemetry-java-instrumentation/releases/latest/download/opentelemetry-javaagent.jar)

Set this environment:
```
export JAVA_TOOL_OPTIONS="-javaagent:/{path_to}/opentelemetry-javaagent.jar"
export OTEL_SERVICE_NAME=netxms
export OTEL_EXPORTER_OTLP_ENDPOINT=http://{opentelemetry_host}:4318  # OTLP endpoint of the collector
export OTEL_EXPORTER_OTLP_PROTOCOL=http/protobuf
export OTEL_TRACES_EXPORTER=otlp
export OTEL_METRICS_EXPORTER=otlp
export OTEL_LOGS_EXPORTER=otlp
```
Restart your app.

Open http://{host}:3000/ 

## Architecture
```
                                                             http/4317  ┌──────────────────┐   http/3200
                                                           ┌----------> | Tempo [ traces ] |<-------------┐
                                                           |            └──────────────────┘              |
┌─────────────────────┐             ┌────────────────┐     | http/3100  ┌────────────────┐ http/3100 ┌─────────┐           
| JAVA app + OT agent | ----------->| Otel Collector | ----└----------> | Loki [ logs ]  |<----------| Grafana |
└─────────────────────┘  http/4318  └───────┬────────┘                  └────────────────┘           └─────────┘ 
                                            ^     pull http/8090        ┌───────────────────────┐        |
                                            └-------------------------- | Prometheus [ metrics] |<-------┘
                                                                        └───────────────────────┘ http/9090
```
Data is stored permanently in local directories grafana-data loki-data prometheus-data tempo-data


 
