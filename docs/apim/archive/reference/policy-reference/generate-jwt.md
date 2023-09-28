---
description: This page provides the technical details of the Generate JWT policy
---

# Generate JWT

## Overview

You use the `generate-JWT` policy to generate a signed JWT with a configurable set of claims. This JWT can subsequently be forwarded to backend targets, or used in some other way.

When a signed JWT is generated, it is put in the `jwt.generated` attribute of the request execution context.

Functional and implementation information for the `generate-JWT` policy is organized into the following sections:

* [Examples](generate-jwt.md#examples)
* [Configuration](generate-jwt.md#configuration)
* [Compatibility Matrix](generate-jwt.md#compatibility-matrix)
* [Errors](generate-jwt.md#errors)
* [Changelogs](generate-jwt.md#changelogs)

## Examples

{% hint style="warning" %}
This policy can be applied to [v2 APIs and v4 proxy APIs.](../../overview/gravitee-api-definitions-and-execution-engines/)

Currently, this policy can **not** be applied at the message level.
{% endhint %}

{% tabs %}
{% tab title="Proxy API example" %}
```
"policy-generate-jwt": {
    "signature":"RSA_RS256",
    "expiresIn":30,
    "expiresInUnit":"SECONDS",
    "issuer":"urn://gravitee-api-gw",
    "audiences":["graviteeam"],
    "customClaims":[],
    "id":"817c6cfa-6ae6-446e-a631-5ded215b404b",
    "content":"-----BEGIN PRIVATE KEY-----\nMIIEvgIBADANBgkqhkiG9w0BAQEFAASCBKgwggSkAgEAAoIBAQDg0MY5LcTnpM/N\nd9ohW/mls6CqF3PoVocwUpKSb324QFuSGvo5s2qzM1JkR2uNTS5lapGltF0Krc5j\nmUgKqVZUx3ie76ngvHTVrz9qNHe9znsTFndtpsaFZuNIiGT8X+eAYgqKUaoKA+3y\nNWynEmXL9ywtFtGommPO1iBwMYfbucuxBmwtklkzxCrFGftAsTJANy8T+CV61TpB\nP2LbFVngfT0uDgjfoG/KMSBUZR88YZNvEyj1mEDPvZPZD6vYUBlTMlWgAwAD+pUn\n6b/a1BsZ69mMvMzvOg9NhuwMLwGDwQ45Gh51Swnzk6a/Oamgpa/ehySfZkypJhPL\ndiutySELAgMBAAECggEBALjo/yFok9wzovfM7I0jqWKxLCS6xYsEII2OXSA0s6Mo\nzCiQJ9/twoVCYTI5zCycntyrmsBAaYavDmK9YJPkVC3HI18WoRNH7pETY4VnQlXL\nz08T24dE9WQkDC1MgkNSXocqHKFIKiOyt7PQXV3NtAzfcGZlrmyPECi/1k5xbt05\nmU1AaM0HAKP5kGmoANEWyaPhYSrShD3EQH8QEjPwrmua62e7kas7x5u5u01tFndv\nG1/rYlApvruwoczBdD3R8WQEdziFn09IcGZUnpBWDkPlEn62qLW8/3k+uF9An9dd\n1c0IoyNopefLvm9W4CXtzFEzJsre32BIutpj66EECAECgYEA+2GYTmd7lVAAMgj/\nMes+HNVqRtg5OiAggx6qvjhi+6hhMLeVKS8mqslMQXewHthbY0+PdyvKRCZnNURj\nUmeZxxk04kOJZqN5ak45NJ6T10PnlZ0vtf2Ym9Mmi4Q29Mzk9SCR9NtVuwRHhGmP\nzOPCXQCwFHeVkqzqkYHIji1ko0sCgYEA5PI5WkWFG/uAPxVZbQreyD1iRgTxEz8B\nn1XefxQ1IV8L5/n48XAgeK1NUbhr4jPSbXL98mX5/RdyCmZORdbPLDRqSVrRepQ3\nAXF82Xp2X9Py/Gn/pIZPXEW54ctnEiW8WVRD2XQ2df1sUq+H5gX/RraiI2O9/CyF\nixZkkC4tIUECgYEAw/lt15HtUpYv0NIawTv4DFqEo/5lft8U+aOq0Oj8ody/CE/W\nxWiw6GxOOquobiOV+3JHEkzdPwwBYhGSrOd/hywrgknMkGvZd/rLti36a9PQc187\nltHBa5nNbu8AORCTXlap8w4bY9UOPDhflwfousCShSJFRTfxFsbrJ4xT7MkCgYBQ\np8TsuHEcWo3jq3HFqH6zrGxinnsPfLLlnyqzOjs9dm6LWtUIuae229bRY1ceaYNI\na6prKuHW99uFLmWE1RhHSm/nR8dkl7KJH6IMO8hYGiMQKYeWPnrW1vmVQkMdcY3Z\nKoZ8pSRKjO0MdCo8LwCvuMeGEC1uGYEybsEeyiW8AQKBgBnkExWeD6KQQL9rrImq\nwhPqz9yuMpIsBtf93fDLXwmy/0VG9L6uDf/3MKl+RYs4PQGe+QQSmXTgqcbHr5ug\nNEFDDK0C9k0Gd0Zl/Z29H6vZWJH9E4ur/xZToeADc3sQT/Ga78LwF8s5EtOPuGVD\nOyCUoLQJgofJWKk2Tp5gKogB\n-----END PRIVATE KEY-----"
}
```
{% endtab %}
{% endtabs %}

## Configuration

### Phases

Policies can be applied to the request or the response of a Gateway API transaction. The request and response are broken up into [phases](broken-reference) that depend on the [Gateway API version](../../overview/gravitee-api-definitions-and-execution-engines/). Each policy is compatible with a subset of the available phases.

The phases checked below are supported by the `generate-JWT` policy:

<table data-full-width="false"><thead><tr><th width="202">v2 Phases</th><th width="139" data-type="checkbox">Compatible?</th><th width="198">v4 Phases</th><th data-type="checkbox">Compatible?</th></tr></thead><tbody><tr><td>onRequest</td><td>true</td><td>onRequest</td><td>true</td></tr><tr><td>onResponse</td><td>false</td><td>onResponse</td><td>false</td></tr><tr><td>onRequestContent</td><td>false</td><td>onMessageRequest</td><td>false</td></tr><tr><td>onResponseContent</td><td>false</td><td>onMessageResponse</td><td>false</td></tr></tbody></table>

### Options

The `generate-JWT` policy can be configured with the following options:

<table><thead><tr><th width="131">Property</th><th width="103" data-type="checkbox">Required</th><th width="210">Description</th><th>Type</th><th>Default</th></tr></thead><tbody><tr><td>signature</td><td>true</td><td>Signature used to sign the token</td><td>Algorithm</td><td>RS256</td></tr><tr><td>kid</td><td>false</td><td>key ID (<code>kid</code>) to include in the JWT header</td><td>string</td><td>-</td></tr><tr><td>id</td><td>false</td><td>JWT ID (<code>jti</code>) claim is a unique identifier for the JWT</td><td>string</td><td>UUID</td></tr><tr><td>audiences</td><td>false</td><td>JWT audience claim; can be a string or an array of strings</td><td>List of string</td><td>-</td></tr><tr><td>issuer</td><td>false</td><td>Claim that identifies the issuer of the JWT</td><td>string</td><td>-</td></tr><tr><td>subject</td><td>false</td><td>Claim that identifies or makes a statement about the subject of the JWT</td><td>string</td><td>-</td></tr></tbody></table>

### Attributes

The `generate-JWT` policy can be configured with the following attributes:

| Name          | Description                 |
| ------------- | --------------------------- |
| jwt.generated | JWT generated by the policy |

You can read the token using the [Gravitee Expression Language](../../guides/policy-design/gravitee-expression-language.md):

```
{#context.attributes['jwt.generated']}
```

## Compatibility matrix

The following is the compatibility matrix for APIM and the `generate-JWT` policy.

<table data-full-width="false"><thead><tr><th>Plugin Version</th><th>Supported APIM versions</th></tr></thead><tbody><tr><td>Up to 1.x</td><td>All</td></tr></tbody></table>

## Errors

<table data-full-width="false"><thead><tr><th width="180">Phase</th><th width="171">HTTP status code</th><th width="387">Message</th></tr></thead><tbody><tr><td>onRequest</td><td><code>500</code></td><td>Unexpected error while creating and signing the token</td></tr></tbody></table>

### Nested objects

To limit the processing time in the case of a nested object, the default max depth of a nested object has been set to 1000. This default value can be overridden using the environment variable `gravitee_policy_jsonxml_maxdepth`.

## Changelogs

{% @github-files/github-code-block url="https://github.com/gravitee-io/gravitee-policy-generate-jwt/blob/master/CHANGELOG.md" %}