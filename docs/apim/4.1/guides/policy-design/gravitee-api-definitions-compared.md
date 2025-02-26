# Gravitee API Definitions Compared

Introduced in release 3.19.0 of the Gravitee API Management platform, there is a new version of a new policy execution engine that enables an improved execution flow for synchronous APIs and supports event-driven policy execution for asynchronous APIs. It is based on a modern and fully reactive architecture designed to address a number of challenges Gravitee users have been facing with the existing policy execution engine, available with the Gravitee’s V2 API definition specification used in versions prior to APIM 3.19.x.

The new policy execution engine builds on the new V4 API definition of Gravitee’s event-native platform. This adds features such as native support for Pub/Sub (Publish-Subscribe) design and enabling policies at message level. The new V4 event-native API definition makes it possible for the Gateway to handle asynchronous API use cases.

The new engine provides the following capabilities:

* The ability to execute policies in the exact order in which they have been placed in Design Studio (see [Create and configure an API flow](https://docs.gravitee.io/apim/3.x/apim\_publisherguide\_design\_studio\_create.html#create\_and\_configure\_an\_api\_flow) and [Configure plan flows and policies](https://docs.gravitee.io/apim/3.x/apim\_publisherguide\_plan\_policies.html)). This addresses some issues experienced by users related to the order in which policies are executed by the V3 engine where policies interacting with the Head part of the request are always executed first, even when placed in a different order in the Design Studio during the design phase. With the new execution engine, it is possible to apply logic on a head policy based on the payload of the request - for example, to apply dynamic routing based on the request payload.
* Proper isolation between platform-level policies and API-level policies during policy execution. This ensures that platform-level policies are always executed prior to any API-level policies during the request stage, and after any API-level policies during the response stage.
* Removal of the need to define a scope for policies (`onRequest`, `onRequestContent`, `onResponse`, `onResponseContent`).
* The introduction of new API types to better differentiate REST APIs from Async APIs (Websocket, SSE, Webhook), as a foundation to fully support the execution of Async APIs.

In this section you can learn about all the differences between the new V4 policy execution engine and the existing V3 execution engine, as well as get guidance on managing changes in system behavior and the potential design and operational impact when switching to the new policy execution engine mode. The information is grouped by functional area in the sub-sections below.

### How to enable and disable the new engine

To enable the new V4 policy execution engine, create a new environment variable called `gravitee_api_jupiterMode_enabled` and set it to `true` on the Management API and the API Gateway.

To disable the new engine and revert to using the V3 engine mode, set this environment variable to `false`.

See [Environment variables](https://docs.gravitee.io/apim/3.x/apim\_installguide\_rest\_apis\_configuration.html#environment\_variables) on the Management API and [Environment variables](https://docs.gravitee.io/apim/3.x/apim\_installguide\_gateway\_configuration.html#environment\_variables) on the API Gateway for more detail about setting environment variables.

### Policy execution phases/scopes and execution order

#### V3 policy execution engine mode

In V3 mode, different execution scopes are required in order to indicate at which level a policy will work, as follows:

* `REQUEST`: The policy only works on request headers. It never accesses the request body.
* `REQUEST_CONTENT`: The policy works at request content level and can access the request body.
* `RESPONSE`: The policy only works on response headers. It never accesses the response body.
* `RESPONSE_CONTENT`: The policy works at response content level and can access the response body.

As a result, all policies working on the body content are postponed to be executed after the policies working on headers. This leads to an execution order than is often different than the one originally designed, as shown in the following diagram:

![event native api management execution scopes 1](https://docs.gravitee.io/images/apim/3.x/event-native/event-native-api-management-execution-scopes-1.png)

#### New V4 policy execution engine mode

In V4 mode, the `REQUEST_CONTENT` and `RESPONSE_CONTENT` scopes (which we now call 'phases') are no longer considered - all policies are executed in the exact order of design, regardless of whether they work on the content or not. This is shown in the following diagram:

![event native api management execution scopes 2](https://docs.gravitee.io/images/apim/3.x/event-native/event-native-api-management-execution-scopes-2.png)

#### Potential impact

If you have designed your APIs with V3 policy execution engine mode ordering in mind, when you switch to using the new V4 mode there may be policy execution behavior changes because the policy execution will be reordered to match the original design policy order.

To smooth the transition process to the new mode, you can use the debug mode to test the new behavior and adapt your APIs, so they can be safely redeployed in the new mode. For example, you can extract some information from the payload to assign it to a header.

In case of any issues or unexpected behavior, you can switch back to V3 execution mode at any time.

### Logging

#### V3 policy execution engine mode

In V3 mode, the following issues exist with logging:

* A 502 status code would normally indicate that the server has responded with a 502 status code, however this is not possible if the connection has failed.
* Consumer response headers are not displayed clearly.

For example:

![event native api management logging 1](https://docs.gravitee.io/images/apim/3.x/event-native/event-native-api-management-logging-1.png)

#### New V4 policy execution engine mode

In V4 mode:

* When a connectivity error occurs during a connection attempt to the backend endpoint, the Gateway response displays an HTTP status code 0 and no headers. This makes it clear that no response has been received from the backend endpoint due to the connectivity error.
* Consumer response headers are displayed more clearly.

For example:

![event native api management logging 2](https://docs.gravitee.io/images/apim/3.x/event-native/event-native-api-management-logging-2.png)

### Condition evaluation

#### V3 policy execution engine mode

In V3 mode, when the Gateway provides a valid EL expression that fails to be evaluated because it is trying to access missing data, the Gateway returns a 500 error with an obscure message.

For example:

![event native api management condition evaluation 1](https://docs.gravitee.io/images/apim/3.x/event-native/event-native-api-management-condition-evaluation-1.png)

#### New V4 policy execution engine mode

In V4 mode, when a valid EL expression is evaluated as `true`, the policy (or flow) is executed. Otherwise, it is skipped.

A policy is skipped when:

* The Expression Language (EL) expression is evaluated as `false`.
* The EL expression evaluation fails because the expected data tested is missing.

This is shown in the example below:

![event native api management condition evaluation 2](https://docs.gravitee.io/images/apim/3.x/event-native/event-native-api-management-condition-evaluation-2.png)

Mastering EL expressions can be difficult, so the new mode ensures that if an EL expression fails because it is trying to access missing data, the condition is evaluated as `false`. This makes the use of EL expressions much simpler and easier.

For example, `{#request.headers['X-Test'][0] == 'something'}` will just skip the execution even if the request header `X-Test` is not specified.

The execution still fails and throws an error if the provided EL expression cannot be parsed (for example, if it is syntactically invalid).

The example below shows the new behavior:

![event native api management condition evaluation 3](https://docs.gravitee.io/images/apim/3.x/event-native/event-native-api-management-condition-evaluation-3.png)

#### Potential impact

In V4 mode, conditions evaluated as errors skip the policy or flow execution instead of returning a 500 error.

As a result, the number of `500` errors should decrease after [activating](https://docs.gravitee.io/apim/3.x/event\_native\_apim\_new\_policy\_execution\_engine\_activation.html) the new V4 policy execution engine mode.

### Connection: close

#### V3 policy execution engine mode

In V3 mode, when a bad request is sent to the Gateway, the Gateway indicates a `Connection: close` response header and effectively closes the connection. The next same call from a client application will then end with the same 400 bad request code and that connection will be closed too. This could happen over and over again as long as the client application sends requests to the Gateway with the same invalid data. The same behavior is in place for 404 "not found" errors.

Creating a connection is costly for the Gateway and such issues can dramatically impact performance - especially if the consumer intensively makes a lot of bad requests.

#### New V4 policy execution engine mode

The new execution engine considers that a bad request does not require to close the connection as it is a client-side error. The engine will only close the connection in case of a server-side error.

#### Potential impact

You can expect decreased CPU consumption in the new mode, especially when a lot of requests end with 4xx errors.

### Flow condition

#### V3 policy execution engine mode

In V3 mode, a condition can be defined once for the whole flow but it is evaluated before executing each phase of the flow (`REQUEST` and `RESPONSE` phases). This could lead to a partial flow execution - for instance, when trying to define a condition based on a request header and this same header is removed during the `REQUEST` phase (for example, the user does not want it to be transmitted to the backend). In such cases, the condition is re-evaluated and the `RESPONSE` phase is skipped completely. The same scenario could happen with a platform flow for the same reasons.

The example below shows this behavior:

![event native api management flow condition 1](https://docs.gravitee.io/images/apim/3.x/event-native/event-native-api-management-flow-condition-1.png)

#### New V4 policy execution engine mode

In V4 mode, the flow condition will be applied once for the whole flow - if the condition is evaluated as `true`, then both the `REQUEST` and the `RESPONSE` phases will be executed.

The example below shows the new behavior:

![event native api management flow condition 2](https://docs.gravitee.io/images/apim/3.x/event-native/event-native-api-management-flow-condition-2.png)

#### Potential impact

If you expect the `RESPONSE` phase to be skipped in the scenario described above, you must refactor your flows since both the `REQUEST` and `RESPONSE` phases will be executed as long as the condition is evaluated as `true`.

To mimic the V3 behavior while executing in the new V4 policy execution engine mode, you can create a new flow and add a condition directly on the policies.

### Flow interruption

#### V3 policy execution engine mode

In V3 mode, when a policy fails, the execution flow is interrupted and the response is returned to the client application. As a result, the platform flow response is also skipped. This leads to unexpected behavior, especially when POST actions are expected (for example, for a custom metrics reporter).

The example below shows this behavior:

![event native api management flow interruption 1](https://docs.gravitee.io/images/apim/3.x/event-native/event-native-api-management-flow-interruption-1.png)

#### New V4 policy execution engine mode

The new V4 policy execution engine ensures that platform flows are always executed (except in case of an irrecoverable error). This allows the API to fail without skipping important steps in the flow occurring at a higher level.

The example below shows the new behavior:

![event native api management flow interruption 2](https://docs.gravitee.io/images/apim/3.x/event-native/event-native-api-management-flow-interruption-2.png)

### Access-Control-Allowed-Origin

#### V3 policy execution engine mode

In V3 mode, when configuring Cross-Origin Resource Sharing (CORS) to allow some origin, the Gateway properly validates the origin but returns `Access-Control-Allowed-Origin: *` in the response header.

#### New V4 policy execution engine mode

In V4 mode, the allowed origin is returned instead of `*` - for example, `Access-Control-Allowed-Origin:` [`https://test.gravitee.io`](https://test.gravitee.io/).

The example below shows the new behavior:

![event native api management cors](https://docs.gravitee.io/images/apim/3.x/event-native/event-native-api-management-cors.png)

### Expression Language (EL) expression parsing

#### V3 policy execution engine mode

In V3 mode, an EL expression is parsed each time it is evaluated.

#### New V4 policy execution engine mode

In V4 mode, a new caching mechanism allows to cache the parsed EL expression for later reuse and therefore to avoid useless parsing of the same expression multiple times.

#### Potential impact

The caching of parsed EL expressions provides for enhanced performance.

### EL expression based on the body

#### V3 policy execution engine mode

In V3 mode, using an EL expression such as `{#request.content == 'something'}` is limited to a few policies working at `REQUEST_CONTENT` or `RESPONSE_CONTENT` - for example, assign metrics, assign content, request validation.

Defining a policy or a flow condition based on request or response body is not supported.

#### New V4 policy execution engine mode

When V4 mode is enabled on an API, it is possible to define a condition based on the body.

It is now possible to execute a complete flow or a policy by applying a condition on the body such as `{#request.content == 'something'}`.

Depending on the expected content type, it is also possible to define a condition based on JSON or XML content such as `{#request.jsonContent.foo.bar == 'something'}` where the request content looks like this:

```
{
	"foo": {
      "bar": "something"
    }
}
```

The same applies for XML content using `{#request.xmlContent.foo.bar == 'something'}`:

```
<foo>
  <bar>something</bar>
</foo>
```

#### Potential impact

Use with caution - using an EL body-based expression is resource-heavy and should be avoided as much as possible. Working with request or response content can significantly degrade performance and consumes substantially more memory on the Gateway.

### Policy support

#### V3 policy execution engine mode

In V3 mode, all existing supported policies will continue to work as before without a change.

Over time, all policies will be migrated to support the new V4 policy execution engine mode. The migration will ensure that all policies are backward compatible with the V3 execution mode throughout the V3 mode’s normal product support life cycle.

#### New V4 policy execution engine mode

The new V4 policy execution engine mode comes with a new Policy interface, which allows you to execute all existing V3-mode policies without the need for any changes.

All policies related to security have already been migrated to support both V3 and V4 execution engine modes, as follows:

* Keyless
* ApiKey
* JWT
* OAuth2

Custom policies developed by community users or customers using V3 mode should be perfectly compatible with the new V4 mode, however we strongly recommend switching to the new V4 mode for custom policy implementation (a developer guide will be published soon).

### Timeout Management

#### V3 policy execution engine mode

In V3 mode, when a timeout is configured (`http.requestTimeout`) and triggered due to a request that is too slow (or a policy taking too much time to execute, such as an HTTP callout policy), the API platform flows are skipped and a 504 status is sent as a response to the client.

#### New V4 policy execution engine mode

In V4 mode, values of `0` and less are treated as meaning 'no timeout' (like in V3 mode). If you configure the timeout with a positive value, then it will act normally.

|   | If no configuration is provided, a default configuration is set to default to 30000 ms timeout. |
| - | ----------------------------------------------------------------------------------------------- |

A timeout can now be triggered on two places in the chain, as follows:

* The flow can be interrupted between the beginning of the request and the end of response API flow. In this case, a platform response flow will be executed.
* The flow can be interrupted during the platform response flow, because the overall request time is too big, causing a 504 response and getting the platform response flow interrupted.

Two properties are available to address this: \* `http.requestTimeout` - the duration used to configure the timeout of the request. \* `http.requestTimeoutGraceDelay` - an additional time used to give the platform response flow a chance to execute.

The timeout value is calculated from the following two properties: \* `Timeout = Max(http.requestTimeoutGraceDelay, http.requestTimeout - apiElapsedTime)` \* With `apiElapsedTime = System.currentTimeMillis() - request().timestamp()`.

**Examples**

|   | In the following examples we assume that there is no timeout defined for the backend in the API’s endpoint configuration. In real life, those timeout values should be shorter than `http.requestTimeout`, and should interrupt the flow at invoker level. |
| - | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |

We will use `http.requestTimeout=2000ms` and `http.requestTimeoutGraceDelay=30ms`.

The example below shows timelines indicating when a timeout should occur depending on the duration of the API flow and the response platform flows:

![event native api management timeout](https://docs.gravitee.io/images/apim/3.x/event-native/event-native-api-management-timeout.png)

### Plan selection

#### Common behavior

The plan selection workflow parses all the published plans in the following order: JWT, OAuth2, ApiKey, Keyless.

The parsed plan is selected for execution if all the following conditions are met: \* The request contains a token corresponding to this plan type (api-key header, authorization header). \* The plan condition rule is either not set or is set incorrectly. \* There is an active subscription matching the incoming request.

#### V3 policy execution engine mode

In V3 mode, the OAuth2 plan **is selected** even if the incoming request does not match a subscription.

No JWT token introspection is done during OAuth2 plan selection.

If there are multiple OAuth2 plans, that would lead to the selection of an irrelevant one.

#### New V4 policy execution engine mode

In V4 mode, the Oauth2 plan **is not selected** if the incoming request does not match a subscription.

During the OAuth2 plan selection, a token introspection is released in order to retrieve the `client_id`, which allows searching for a subscription.

Due to concerns about performance, a cache system is available to avoid doing the same token introspection multiple times.

Where possible, it is recommended to use selection rules if there are multiple OAuth2 plans, in order to avoid any unnecessary token introspection.

|   | The policy has been changed for the Keyless plan - its activation is now prevented in case a security token has been detected in the incoming request by one of the previous plans. Therefore, if an API has multiple plans (JWT, OAuth2, Apikey, Keyless) and the incoming request contains a token or an apikey that does not match any of the existing plans, then the Keyless plan will not be activated and the user will receive a generic 401 response without any details. |
| - | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |

\
