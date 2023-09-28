---
description: This page provides the technical details of the JWT policy
---

# JSON Web Token (JWT)

## Overview

You can use the `jwt` policy to validate the token signature and expiration date before sending the API call to the target backend.

Some authorization servers use OAuth2 protocol to provide access tokens. These access token can be in JWS/JWT format. For the RFC standards, see:

* JWS (JSON Web Signature) standard RFC: [https://tools.ietf.org/html/rfc7515](https://tools.ietf.org/html/rfc7515)
* JWT (JSON Web Token) standard RFC: ([https://tools.ietf.org/html/rfc7519](https://tools.ietf.org/html/rfc7519)

Functional and implementation information for the `jwt` policy is organized into the following sections:

* [Examples](json-web-token-jwt.md#examples)
* [Configuration](json-web-token-jwt.md#configuration)
* [Compatibility Matrix](json-web-token-jwt.md#compatibility-matrix)
* [Errors](json-web-token-jwt.md#errors)
* [Changelogs](json-web-token-jwt.md#changelogs)

## Examples

{% hint style="warning" %}
This policy can be applied to [v2 APIs and v4 proxy APIs.](../../overview/gravitee-api-definitions-and-execution-engines/) Currently, this policy can **not** be applied at the message level.
{% endhint %}

{% tabs %}
{% tab title="Proxy API example" %}
Given the following JWT claims (payload):

```
{
  "iss": "Gravitee.io AM",
  "sub": "1234567890",
  "name": "John Doe",
  "admin": true
}
```

You can extract the issuer from JWT using the following Expression Language statement:

```
{#context.attributes['jwt.claims']['iss']}
```
{% endtab %}
{% endtabs %}

## Configuration

To validate the token signature, the policy needs to use the associated Authorization Servers public key.

The policy prompts you to choose between three (`GIVEN_KEY`, `GIVEN_ISSUER`, `GATEWAY_ISSUER`) methods to retrieve the required public key.

* `GIVEN_KEY` — You provide the key (in `ssh-rsa`, `pem`, `crt` or `public-key` format)
* `GIVEN_ISSUER` — If you want to filter on several authorization servers then you only need to specify the issuer name; the gateway will only accept JWTs with a permitted issuer attribute. If `GATEWAY_KEYS` is set, the issuer is also used to retrieve the public key from the `gravitee.yml` file.
* `GATEWAY_KEYS` — You can set some public keys in the APIM Gateway `gravitee.yml` file

```
policy:
  jwt:
    issuer:
      my.authorization.server:
        default: ssh-rsa myValidationKey anEmail@domain.com
        kid-2016: ssh-rsa myCurrentValidationKey anEmail@domain.com
```

The policy will inspect the JWT:

* Header to extract the key id (`kid` attribute) of the public key. If no key id is found then it use the `x5t` field.
  * If `kid` is present and no key corresponding is found, the token is rejected.
  * If `kid` is missing and no key corresponding to `x5t` is found, the token is rejected.
* Claims (payload) to extract the issuer (`iss` attribute).

Using these two values, the Gateway can retrieve the corresponding public key.

Regarding the `client_id`, the standard behavior is to read it from the `azp` claim, then if not found in the `aud` claim and finally in the `client_id` claim. You can override this behavior by providing a custom `clientIdClaim` in the configuration.

### Phases

The phases checked below are supported by the `jwt` policy:

<table data-full-width="false"><thead><tr><th width="202">v2 Phases</th><th width="120" data-type="checkbox">Compatible?</th><th width="203.41136671177264">v4 Phases</th><th data-type="checkbox">Compatible?</th></tr></thead><tbody><tr><td>onRequest</td><td>true</td><td>onRequest</td><td>true</td></tr><tr><td>onResponse</td><td>false</td><td>onResponse</td><td>false</td></tr><tr><td>onRequestContent</td><td>false</td><td>onMessageRequest</td><td>false</td></tr><tr><td>onResponseContent</td><td>false</td><td>onMessageResponse</td><td>false</td></tr></tbody></table>

### Options

The `jwt` policy can be configured with the following options:

<table><thead><tr><th width="194">Property</th><th width="105" data-type="checkbox">Required</th><th width="224">Description</th><th>Type</th><th>Default</th></tr></thead><tbody><tr><td>publicKeyResolver</td><td>true</td><td>Used to resolve the public key needed to validate the signature</td><td>enum</td><td>GIVEN_KEY</td></tr><tr><td>resolverParameter</td><td>false</td><td>Needed if you use the <code>GATEWAY_KEYS</code> or <code>GIVEN_ISSUER</code> resolver (EL support)</td><td>string</td><td></td></tr><tr><td>useSystemProxy</td><td>false</td><td>Select this option if you want use system proxy (only useful when resolver is <code>JWKS_URL</code>)</td><td>boolean</td><td>false</td></tr><tr><td>extractClaims</td><td>false</td><td>Select this option if you want to extract claims into the request context</td><td>boolean</td><td>false</td></tr><tr><td>clientIdClaim</td><td>false</td><td>Required if the client_id should be read from non-standard claims (azp, aud, client_id)</td><td>string</td><td></td></tr></tbody></table>

### Attributes

The `jwt` policy can be configured with the following attributes:

| Name       | Description                                                                                                                                         |
| ---------- | --------------------------------------------------------------------------------------------------------------------------------------------------- |
| jwt.token  | JWT token extracted from the `Authorization` HTTP header                                                                                            |
| jwt.claims | A map of claims registered in the JWT token body, used for extracting data from it. Only if `extractClaims` is enabled in the policy configuration. |

## Compatibility matrix

The following is the compatibility matrix for APIM and the `jwt` policy:

| Plugin version   | Supported APIM versions |
| ---------------- | ----------------------- |
| 4.x+             | 4.0.x+                  |
| 2.x+             | 3.18.x to 3.20          |
| 1.22.x+          | 3.15.x to 3.17.x        |
| 1.20.x to 1.21.x | 3.10.x to 3.14.x        |
| Up to 1.19.x     | Up to 3.9.x             |

## Errors

<table data-full-width="false"><thead><tr><th width="205.5">HTTP status code</th><th width="387">Error template key</th></tr></thead><tbody><tr><td><code>401</code></td><td>Bad token format, content, signature, expired token or any other issue preventing the policy from validating the token</td></tr></tbody></table>

You can use the response template feature to override the default response provided by the policy. These templates must be defined at the API level (see the API Console **Response Templates** option in the API **Proxy** menu).

The error keys sent by the policy are as follows:

| Key                 | Parameters |
| ------------------- | ---------- |
| JWT\_MISSING\_TOKEN | -          |
| JWT\_INVALID\_TOKEN | -          |

## Changelogs

{% @github-files/github-code-block url="https://github.com/gravitee-io/gravitee-policy-jwt/blob/master/CHANGELOG.md" %}