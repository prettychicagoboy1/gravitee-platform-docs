---
description: This page provides the technical details of the Resource Filtering policy
---

# Resource Filtering

## Overview

Functional and implementation information for the Resource Filtering policy is organized into the following sections:

* [Configuration](template-policy-rework-structure-25.md#configuration)
* [Compatibility](template-policy-rework-structure-25.md#compatibility-matrix)
* [Errors](template-policy-rework-structure-25.md#errors)
* [Changelogs](template-policy-rework-structure-25.md#changelogs)

You can use the `resource-filtering` policy to filter REST resources. By applying this filter, you can restrict or allow access to a specific resource determined by a path and a method (or an array of methods).

This policy is mainly used in plan configuration, to limit subscriber access to specific resources only.

A typical usage would be to allow access to all paths (`/**`) but in read-only mode (GET method).



{% hint style="warning" %}
This example will work for [v2 APIs and v4 proxy APIs.](../../overview/gravitee-api-definitions-and-execution-engines.md)

Currently, this policy can **not** be applied at the message level.
{% endhint %}

{% hint style="info" %}
You can’t apply whitelisting and blacklisting to the same resource. Whitelisting takes precedence over blacklisting.
{% endhint %}

## Configuration

Policies can be added to flows that are assigned to an API or to a plan. Gravitee supports configuring policies [through the Policy Studio](../../guides/policy-design/) in the Management Console or interacting directly with the Management API.

When using the Management API, policies are added as flows either directly to an API or to a plan. To learn more about the structure of the Management API, check out the [reference documentation here.](../management-api-reference/)

{% code title="Sample Configuration" %}
```json
"resource-filtering" : {
    "whitelist":[
        {
            "pattern":"/**",
            "methods": ["GET"]
        }
    ]
}
```
{% endcode %}

#### **Ant style path pattern**

URL mapping matches URLs using the following rules:

* `?` matches one character
* `*` matches zero or more characters
* `**` matches zero or more directories in a path

### Reference

<table><thead><tr><th>Property</th><th data-type="checkbox">Required</th><th>Description</th><th>Type</th><th>Default</th></tr></thead><tbody><tr><td>whitelist</td><td>false</td><td>List of allowed resources</td><td>array of <a href="https://docs.gravitee.io/apim/3.x/apim_policies_resource_filtering.html#gravitee-policy-resource-filtering-resource"><code>resources</code></a></td><td>-</td></tr><tr><td>blacklist</td><td>false</td><td>List of restricted resources</td><td>array of <a href="https://docs.gravitee.io/apim/3.x/apim_policies_resource_filtering.html#gravitee-policy-resource-filtering-resource"><code>resources</code></a></td><td>-</td></tr></tbody></table>

A resource is defined as follows:

<table><thead><tr><th>Property</th><th data-type="checkbox">Required</th><th>Description</th><th>Type</th><th>Default</th></tr></thead><tbody><tr><td>pattern</td><td>true</td><td>An <a href="https://docs.gravitee.io/apim/3.x/apim_policies_resource_filtering.html#gravitee-policy-resource-filtering-ant">Ant-style path patterns</a> (<a href="http://ant.apache.org/">Apache Ant</a>).</td><td>string</td><td>-</td></tr><tr><td>methods</td><td>false</td><td>List of HTTP methods for which filter is applied.</td><td>array of HTTP methods</td><td>All HTTP methods</td></tr></tbody></table>

### Phases

Policies can be applied to the request or the response of a Gateway API transaction. The request and response are broken up into [phases](broken-reference) that depend on the [Gateway API version](../../overview/gravitee-api-definitions-and-execution-engines.md). Each policy is compatible with a subset of the available phases.

The phases checked below are supported by the JSON-to-XML policy:

<table data-full-width="false"><thead><tr><th width="209">v2 Phases</th><th width="139" data-type="checkbox">Compatible?</th><th width="188.41136671177264">v4 Phases</th><th data-type="checkbox">Compatible?</th></tr></thead><tbody><tr><td>onRequest</td><td>false</td><td>onRequest</td><td>true</td></tr><tr><td>onResponse</td><td>false</td><td>onResponse</td><td>true</td></tr><tr><td>onRequestContent</td><td>true</td><td>onMessageRequest</td><td>true</td></tr><tr><td>onResponseContent</td><td>true</td><td>onMessageResponse</td><td>true</td></tr></tbody></table>

## Compatibility matrix

The [changelog for each version of APIM](../../releases-and-changelog/changelog/) provides a list of policies included in the default distribution. The chart below summarizes this information in relation to the `json-xml` policy.

<table data-full-width="false"><thead><tr><th width="161.33333333333331">Plugin Version</th><th width="242">Supported APIM versions</th><th>Included in APIM default distribution</th></tr></thead><tbody><tr><td>2.2</td><td>>=3.20</td><td>>=3.21</td></tr><tr><td>2.1</td><td>^3.0</td><td>>=3.0 &#x3C;3.21</td></tr><tr><td>2.0</td><td>^3.0</td><td>N/a</td></tr></tbody></table>

## Errors

#### HTTP status codes

| Code  | Message                                                                   |
| ----- | ------------------------------------------------------------------------- |
| `403` | Access to the resource is forbidden according to resource-filtering rules |
| `405` | Method not allowed while accessing this resource                          |

#### Default response override

You can use the response template feature to override the default responses provided by the policy. These templates must be defined at the API level (see the API Console **Response Templates** option in the API **Proxy** menu).

#### Error keys

The error keys sent by this policy are as follows:

| Key                                       | Parameters    |
| ----------------------------------------- | ------------- |
| RESOURCE\_FILTERING\_FORBIDDEN            | path - method |
| RESOURCE\_FILTERING\_METHOD\_NOT\_ALLOWED | path - method |

## Changelogs

{% @github-files/github-code-block url="https://github.com/gravitee-io/gravitee-policy-resource-filtering/blob/master/CHANGELOG.md" %}