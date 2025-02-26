---
description: Learn how to configure the internal Gateway API
---

# Internal API

## Introduction

The Gravitee APIM Gateway component comes with its own internal API, for monitoring and retrieving technical information about the component.

## Configuration

You need to enable the API as a service in the `gravitee.yaml` file and update any other required configuration.

```
services:
  core:
    http:
      enabled: true
      port: 18082
      host: localhost
      authentication:
        type: basic
        users:
          admin: adminadmin
```

The above values can be understood as such:

`enabled`: whether the service is enabled (default `true`).

`port`: the port the service listens on (default `18082`). You must ensure you use a port which is not already in use by another APIM component.

`host`: the host (default `localhost`).

`authentication.type`: the authentication type for requests:

* `none` if no authentication is required or `basic` (default `basic`).

`authentication.users`: a list of `user: password` combinations. Only required if authentication type is `basic`.

### Endpoints

<table data-full-width="true"><thead><tr><th width="201">Operation</th><th width="320.3333333333333">Description</th><th>Example</th></tr></thead><tbody><tr><td><pre data-overflow="wrap"><code>GET /_node
</code></pre></td><td>Gets generic node information</td><td><pre data-overflow="wrap"><code>HTTP/1.1 200 OK
Content-Type: application/json
{
    "id": "a70b9fd9-9deb-4ccd-8b9f-d99deb6ccd32",
    "metadata": {},
    "name": "Gravitee.io - Management API",
    "version": {
        "BUILD_ID": "309",
        "BUILD_NUMBER": "309",
        "MAJOR_VERSION": "1.20.14",
        "REVISION": "132e719ef314b40f352e6399034d68a9a95e95ef"
    }
}
</code></pre></td></tr><tr><td><pre data-overflow="wrap"><code>GET /_node/health?probes=#probe1,#probe2
</code></pre></td><td><p>Gets the health status of the component. Probes can be filtered using the optional <code>probes</code> query param. The parameter can handle a list of probes, separated by commas (<code>,</code>). If no query param, you get the health of all probes. If the return status is 200 then everything is ok, if 500, there is at least one error. This endpoint can be used by a load balancer, to determine if a component instance is not in the pool, for example.</p><p>Some probes are not displayed by default. You have to explicitly use the query param to retrieve them. These probes are:</p><p>- <strong>cpu</strong></p><p>- <strong>memory</strong></p><p>- <strong>api-sync</strong></p><p>Those probes are considered healthy if there are under a configurable threshold (default is 80%). To configure it, add in your <code>gravitee.yml</code>:</p><p>[source, yml] ---- services: health: threshold: cpu: 80 memory: 80 ----</p></td><td><pre><code>HTTP/1.1 200 OK
Content-Type: application/json
{
"management-api": {
"healthy": true
},
"management-repository": {
"healthy": true
},
"sync-process": {
"healthy": true
},
"sync-process": {
"healthy": true
}
}
</code></pre></td></tr><tr><td></td><td></td><td></td></tr><tr><td><pre data-overflow="wrap"><code>GET /_node/configuration
</code></pre></td><td>Gets the node configuration from the <code>gravitee.yml</code> file and/or environment variables.</td><td><pre><code><strong>HTTP/1.1 200 OK
</strong>Content-Type: application/json
{
"analytics.elasticsearch.endpoints[0]": "http://${ds.elastic.host}:${ds.elastic.port}",
"analytics.type": "elasticsearch",
"ds.elastic.host": "localhost",
"ds.elastic.port": 9200,
...
}
</code></pre></td></tr><tr><td><pre class="language-sh" data-overflow="wrap"><code class="lang-sh">GET /_node/monitor
</code></pre></td><td>Gets monitoring information from the JVM and the server.</td><td><pre><code>HTTP/1.1 200 OK
Content-Type: application/json
{
"jvm": {
"gc": {
"collectors": [
{
"collectionCount": 7,
"collectionTime": 98,
"name": "young"
},
{
"collectionCount": 3,
"collectionTime": 189,
"name": "old"
}
]
},
"mem": {
...
}
</code></pre></td></tr></tbody></table>

### Component-specific endpoints

In addition to the main endpoints listed above, the API includes dedicated endpoints to get more information about the APIs deployed on the APIM Gateway instance.

<table data-full-width="true"><thead><tr><th width="227">Operation</th><th width="220.66666666666669">Description</th><th>Example</th></tr></thead><tbody><tr><td><pre data-overflow="wrap"><code>GET /_node/apis
</code></pre></td><td>Gets the APIs deployed on this APIM Gateway instance.</td><td><pre><code>HTTP/1.1 200 OK
Content-Type: application/json
[
{
"id": "5b7a30b5-8feb-4c11-ba30-b58feb6c112f",
"name": "Foo API",
"version": "1.0.0"
},
{
"id": "5da639b6-a3c7-4cc5-a639-b6a3c75cc5f9",
"name": "Bar API",
"version": "v1"
}
]
</code></pre></td></tr><tr><td><pre data-overflow="wrap"><code>GET /_node/apis/{api.id}
</code></pre></td><td>Gets the API configuration for this APIM Gateway instance.</td><td><pre><code>HTTP/1.1 200 OK
Content-Type: application/json
{
"deployedAt": 1552574408611,
"enabled": true,
"id": "5b7a30b5-8feb-4c11-ba30-b58feb6c112f",
"name": "Foo API",
"pathMappings": {},
"paths": {
...
}
</code></pre></td></tr></tbody></table>
