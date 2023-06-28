---
description: An overview of hybrid deployments
---

# Hybrid Deployment

## Introduction

Hybrid architectures (i.e. a combination of on-premise and cloud deployments) present unique technical and/or cost constraints when it comes to deploying Gravitee API Management (APIM) components in different data centers.

The APIM hybrid deployment solution (in the form of hybrid components, or plugins) overcomes these constraints, giving you the freedom to define your architecture and deployment however you want.

## Architecture

The following diagram shows a typical hybrid APIM architecture:

<figure><img src="https://docs.gravitee.io/images/apim/3.x/installation/hybrid/hybrid_deployment_architecture.png" alt=""><figcaption><p>Hybrid deployment architecture</p></figcaption></figure>

## Configuration

For the APIM Gateway to work in this setup, you need two components:

* A _bridge_ API Gateway (shown in green in the diagram) that exposes extra HTTP services for bridging HTTP calls to the underlying repository (which can be any of our supported repositories: MongoDB, JDBC, and so on)
* A _standard_ APIM Gateway (shown in red on the schema) with the default repository plugin switched to a new HTTP bridge repository plugin

In this infrastructure, the standard APIM Gateway can communicate with the bridge API Gateway through a secure HTTP/S channel, and your cloud data center does not need to have a datastore installed.

{% hint style="info" %}
What we describe as a _bridge_ API Gateway is, in fact, a standard APIM Gateway augmented with a new plugin.
{% endhint %}

## APIM Gateway - HTTP bridge (server)

### **Basic installation**

To expose the new HTTP API, you need to install a new plugin inside the `plugins` directory of APIM Gateway. This plugin can be found [here](https://download.gravitee.io/#graviteeio-apim/plugins/repositories/gravitee-apim-repository-gateway-bridge-http-server/).

{% hint style="info" %}
This plugin is disabled by default from APIM 3.13.0.
{% endhint %}

{% code overflow="wrap" %}
```sh
wget -O ${GRAVITEEIO_HOME}/plugins https://download.gravitee.io/graviteeio-apim/plugins/repositories/gravitee-apim-repository-gateway-bridge-http-server/gravitee-apim-repository-gateway-bridge-http-server-${PLUGIN_VERSION}.zip
```
{% endcode %}

{% hint style="info" %}
You can safely remove all plugins that start with the following names - these are not used on a bridge server gateway but are available by default in the APIM Gateway:

* gravitee-apim-gateway-services-sync\*
* gravitee-policy-\*
* gravitee-resource-\*
{% endhint %}

### **Basic configuration**

You configure the new plugin in the `gravitee.yaml` file.

{% code title="gravitee.yml" %}
```yaml
services:
  bridge:
    http:
      enabled: true
      port: 18092
      host: localhost
      authentication:
        # authentication type to be used for the core services
        # - none : to disable authentication
        # - basic : to use basic authentication
        # default is "basic"
        type: basic
        users:
          admin: adminadmin
      secured: true
      ssl:
        clientAuth: false
        keystore:
          type: # can be jks / pem / pkcs12
          path:
          password:
          certs: # Required for pem
            -  /path/to/cert
          keys:
            -  /path/to/key
        trustore:
          type: # can be jks / pem / pkcs12
          path:
          password:
```
{% endcode %}

### **Check the APIM Gateway (HTTP bridge server) node is running**

You can test that your APIM Gateway (HTTP bridge server) node is running by sending an HTTP request to port `18092` on `localhost`:

```sh
curl -X GET http://localhost:18092/_bridge/apis
```

You should receive a response containing an empty array or a list of APIs.

## APIM Gateway - HTTP repository (client)

### **Basic installation**

To consume the HTTP bridge, you need to replace default repository plugins (usually a MongoDB repository) with a new HTTP repository in the APIM Gateway `plugins` directory. This plugin can be found [here](https://download.gravitee.io/#graviteeio-apim/plugins/repositories/gravitee-apim-repository-gateway-bridge-http-client/).

{% code overflow="wrap" %}
```sh
wget -O ${GRAVITEEIO_HOME}/plugins https://download.gravitee.io/graviteeio-apim/plugins/repositories/gravitee-apim-repository-gateway-bridge-http-client/gravitee-apim-repository-gateway-bridge-http-client-${PLUGIN_VERSION}.zip
```
{% endcode %}

### **Basic configuration**

You configure the new plugin in the `gravitee.yaml` file in the APIM Gateway `config` directory.

{% code title="gravitee.yaml" %}
```yaml
management:
  type: http
  http:
    url: http://localhost:18092/
    keepAlive: true
    idleTimeout: 30000
    connectTimeout: 10000
    authentication:
      basic:
        username: admin
        password: adminadmin
    ssl:
      trustAll: true
      verifyHostname: true
      keystore:
        type: # can be jks / pem / pkcs12
        path:
        password:
      trustore:
        type: # can be jks / pem / pkcs12
        path:
        password:
```
{% endcode %}

## Start the APIM Gateways

Start both APIM Gateways. Your consumers will be able to call the APIM Gateway with the HTTP repository as in a standard deployment.