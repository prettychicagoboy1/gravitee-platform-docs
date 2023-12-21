# CIBA

## Overview

[The Client-Initiated Backchannel Authentication Flow - Core 1.0 (CIBA)](https://openid.net/specs/openid-client-initiated-backchannel-authentication-core-1\_0.html) is an authentication flow where the Relying Party communicates with OpenID Provider without redirects through the user’s browser.

## Protocol

The purpose of the Client-Initiated Backchannel Authentication Flow (CIBA) is to authenticate a user without relying on browser redirections. With this flow, the Relying Parties (RPs), that can obtain a valid identifier for the user they want to authenticate, will be able to initiate an interaction flow to authenticate their users without having end-user interaction from the consumption device. The flow involves direct communication from the Client to the OpenID Provider (OP) without redirecting through the user’s browser (consumption device). In order to authenticate the user, the OP will initiate an interaction with an Authentication Device (AD) like a smartphone.

To activate CIBA on your security domain:

* Click **Settings > CIBA**
* Switch on the **Enable CIBA** toggle
* Adapt the CIBA Settings if necessary
* Save your choice

### CIBA settings

There are three parameters for CIBA:

* The validity of the **auth\_req\_id** in second. The **auth\_req\_id** is generated by the [Authentication Backchannel Endpoint](https://openid.net/specs/openid-client-initiated-backchannel-authentication-core-1\_0.html#auth\_backchannel\_endpoint) in order to request a token once the user has been authenticated with the Authentication Device.
* The interval in seconds that a client must wait between two calls on the token endpoint to obtain an **access\_token** using a given **auth\_req\_id**.
* The maximum number of characters allowed for the **binding\_message** parameter.

The plugin is used to manage the Authentication Device interaction.

### Authentication device interaction plugins

In order to manage the interactions with the user devices, AM comes with a plugin mechanism to select the implementation that feat your needs. See the list of available [plugins](ciba.md#authentication-device-plugins) for more details.

## Client Registration

In order to provide a client configuration compatible with CIBA, the client has to register using the [Dynamic Client Registration](https://openid.net/specs/openid-connect-registration-1\_0.html) endpoint.

For more information about the parameters related to CIBA, please see the [Registration and Discovery Metadata](https://openid.net/specs/openid-client-initiated-backchannel-authentication-core-1\_0.html#registration) section of the specification.

An example of payload for a client following CIBA.

{% code overflow="wrap" %}
```sh
{
        "redirect_uris": ["https://mybank.com/callback"],
        "client_name": "client1",
        "application_type" : "web",
        "grant_types": [ "authorization_code","refresh_token"],
        "response_types" : [
                "code",
                "code id_token token",
                "code id_token",
                "code token"
        ],
        "scope":"openid payments",
        "jwks_uri": "https://mybank.com/.well-known/jwks_uri.json",
        "default_acr_values" : ["urn:mace:incommon:iap:silver"],
        "authorization_signed_response_alg" : "PS256",
        "id_token_signed_response_alg" : "PS256",
        "request_object_signing_alg" : "PS256",
        "token_endpoint_auth_method" : "tls_client_auth",
        "tls_client_auth_subject_dn": "C=FR, ST=France, L=Lille, O=mybank, OU=Client1, CN=mycompamybankgny.com, EMAILADDRESS=contact@mybank.com",
        "tls_client_certificate_bound_access_tokens": true,
        "backchannel_token_delivery_mode": "poll",
        "backchannel_authentication_request_signing_alg": "PS256",
        "backchannel_user_code_parameter": false
      }'
```
{% endcode %}

## Hints

The [authentication request](https://openid.net/specs/openid-client-initiated-backchannel-authentication-core-1\_0.html#auth\_request) exposes 3 parameters in order to identify the end user: `login_hint`, `login_hint_token` and `id_token_hint`. The `id_token_hint` is the standard ID Token JWT so the `sub` claim will be used to identify the end user. The `login_hint` is a simple string value, AM only considers this parameter as representing the **username** or the user **email address**. Finally, the `login_hint_token` is an unspecified JWT that contains information that aims to identify the end-user. In order to manage this parameter, AM accepts the following payload for this JWT where:

* `format` specify the attribute used to identify the end-user (possible values are `username` and `email`)
* According to the format the second entry will be either `username` or `email` with the associated value.

```json
{
  "sub_id": {
    "format": "email",
    "email": "myuser@acme.com"
  }
}
```

## Authentication device plugins

The goal of CIBA is to avoid browser redirects in order to grab the user's authorization or identity. The common way to obtain this is to rely on the smartphone of the end user by sending a push notification on a mobile app.

This page introduces AM plugins that will allow you to manage these device notifications.

### External HTTP Service

The **External HTTP Service** plugin brings you the freedom of implementing the notification mechanism in the way you want to by delegating this responsibiltiy to an external HTTP service.

This service must follow the requirements hereafter :

* Be registered as an application on AM in order to provide client ID and client Secret on the AM callback endpoint
* Implement the [notification endpoint](https://docs.gravitee.io/am/current/ciba\_external\_service/index.html) to receive a notification request
* Call the AM [callback endpoint](https://docs.gravitee.io/am/current/ciba/index.html) to update the authentication request status

\\

<figure><img src="https://docs.gravitee.io/images/am/current/graviteeio-am-CIBA-Flow.png" alt=""><figcaption><p>External HTTP service example</p></figcaption></figure>