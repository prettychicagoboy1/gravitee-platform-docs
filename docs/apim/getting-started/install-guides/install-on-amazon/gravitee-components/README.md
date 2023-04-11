# APIM Full Stack Installation

This section describes how to install the full APIM stack and its dependencies at once.&#x20;

Alternatively, you can install the components individually as detailed on the [APIM Components Installation page. ](../apim-components-installation.md)

## Prerequisites

* Amazon instance running
* Gravitee `yum` repository added
* Java 11 JRE installed
* MongoDB installed and running
* Elasticsearch installed and running
* Nginx installed

## Security group

* open port 8082
* open port 8083
* open port 8084
* open port 8085

## Instructions

1. Install all Gravitee APIM components:

```sh
sudo yum install graviteeio-apim-3x -y
```

2. Enable gateway and management API on startup:

```sh
sudo systemctl daemon-reload

sudo systemctl enable graviteeio-apim-gateway

sudo systemctl enable graviteeio-apim-rest-api
```

3. Start gateway and management API:

```sh
sudo systemctl start graviteeio-apim-gateway

sudo systemctl start graviteeio-apim-rest-api
```

4. Restart Nginx:

```sh
sudo systemctl restart nginx
```

5. Verify, if any of the prerequisites are missing, you will receive errors during this step:

```sh
sudo journalctl -f
```

{% hint style="info" %}
You can see the same logs in `/opt/graviteeio/apim/gateway/logs/gravitee.log` and `/opt/graviteeio/apim/rest-api/logs/gravitee.log`
{% endhint %}

6. Additional verification:

```sh
sudo ss -lntp '( sport = 8082 )'

sudo ss -lntp '( sport = 8083 )'

sudo ss -lntp '( sport = 8084 )'

sudo ss -lntp '( sport = 8085 )'
```

You should see that there are processes listening on those ports.

7. Final verification:

```sh
curl -X GET http://localhost:8082/

curl -X GET http://localhost:8083/management/organizations/DEFAULT/console

curl -X GET http://localhost:8083/portal/environments/DEFAULT/apis
```

If the installation was successful, then the first API call returns: **No context-path matches the request URI.** The final two API calls should return a JSON payload in the response.

{% hint style="success" %}
Congratulations! Now that APIM is up and running, check out the [Tutorials](../../../tutorials/) for your next steps.
{% endhint %}