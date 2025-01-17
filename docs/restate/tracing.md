---
sidebar_position: 4
description: "Find out how to export traces of your invocations."
---

# Tracing

Restate supports the following tracing features:

* Runtime execution tracing per invocation
* Exporting traces to OTLP-compatible systems (e.g. Jaeger)
* Correlating parent traces of incoming gRPC/Connect HTTP requests, using the [W3C TraceContext](https://github.com/w3c/trace-context) specification.

## Setting up OTLP exporter

Set up the [OTLP exporter](https://github.com/open-telemetry/opentelemetry-collector/blob/main/exporter/otlpexporter/README.md) by pointing the configuration entry `observability.tracing.endpoint` to your trace collector.
The exporter sends OTLP trace data via gRPC.


## Exporting OTLP traces to Jaeger

Jaeger accepts OTLP trace data via gRPC on port `4317`.

:::note
Start Jaeger with the environment variable `COLLECTOR_OTLP_ENABLED=true` to enable OTLP.
:::

Refer to the [Jaeger documentation](https://www.jaegertracing.io/docs/1.46/deployment/) for more details on how to deploy Jaeger.

:::note
Configure the tracing endpoint in Restate as a fully specified URL: `http://<jaeger-collector>:4317`.
:::

You can configure a span/event filter in a similar fashion to the [log filter](/restate/logging#log-filter) setting the `observability.tracing.filter` configuration entry.

#### Try it out locally
To experiment with this feature locally, run the Jaeger container via:
```shell
docker run -d --name jaeger \
-e COLLECTOR_OTLP_ENABLED=true \
-p 4317:4317 \
-p 16686:16686 \
jaegertracing/all-in-one:1.46
```

Then launch Restate with the tracing endpoint defined as an environment variable:
<Tabs groupId="operating-systems">
<TabItem value="lin" label="Linux">

```shell
docker run --name restate_dev --rm -d -e RESTATE_OBSERVABILITY__TRACING__ENDPOINT=http://localhost:4317 --network=host ghcr.io/restatedev/restate-dist:VAR::RESTATE_DIST_VERSION
```

</TabItem>
<TabItem value="mac" label="macOS">

```shell
docker run --name restate_dev --rm -d -e RESTATE_OBSERVABILITY__TRACING__ENDPOINT=http://host.docker.internal:4317 -p 8081:8081 -p 9091:9091 -p 9090:9090 ghcr.io/restatedev/restate-dist:VAR::RESTATE_DIST_VERSION
```

</TabItem>
</Tabs>

Go to the Jaeger UI at http://localhost:16686.

If you now spin up your Restate services and send requests to them, you will see the traces appear in the Jaeger UI.

An example from the [shopping cart example](https://github.com/restatedev/examples/tree/main/typescript/ecommerce-store):
![Observability](/img/observability.jpeg)

## Setting up Jaeger file exporter

If you can't configure a Jaeger agent, you can export traces by writing them to files, using the Jaeger JSON format. To do so, set up the configuration entry `observability.tracing.json_file_export_path` pointing towards the path where trace files should be written.

You can import the trace files using the Jaeger UI:

![Jaeger UI File import](/img/jaeger-import-file.png)


## Understanding traces
The traces contain detailed information about the context calls that were done during the invocation (e.g. sleep, one-way calls, interaction with state):

![Understanding traces](/img/jaeger_tour_background_call_handler.png)

The initial `ingress_invoke` spans show when the gRPC/Connect HTTP request was received by Restate. The `invoke` span beneath it shows when Restate invoked the service endpoint to process the request.

The tags of the spans contain the metadata of the context calls (e.g. call arguments, invocation id). 

When a service invokes another service, the child invocation is linked automatically to the parent invocation, as you can see in the image.
Note that the spans of one-way calls are shown as separate traces. The parent invocation only shows that the one-way call was scheduled, not its entire tracing span. 
To see this information, search for the trace of the one-way call by filtering on the invocation id tag:
```
restate.invocation.id="T4pIkIJIGAsBiiGDV2dxK7PkkKnWyWHE"
```

## Searching traces

Traces export attributes and tags that correlate the trace with the service and/or invocation. For example, in the Jaeger UI, you can filter on the invocation id (`restate.invocation.id`) or any other tag:

![Jaeger invocation id search](/img/jaeger_invocationid_search_handler.png)