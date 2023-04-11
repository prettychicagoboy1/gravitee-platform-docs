# Gravitee Kubernetes Operator

The Gravitee Kubernetes Operator (GKO) - a technical component designed to be deployed on an existing APIM-ready Kubernetes Cluster. It can also be deployed on a local cluster for testing purposes.

You can use the GKO to define, deploy, and publish APIs to your API Portal and API Gateway and to manage Custom Resource Definitions (CRDs) as part of the process.

The GKO also enables you to create reusable [API resources](https://docs.gravitee.io/apim/3.x/apim\_resources\_overview.html) by applying the [`ApiResource` custom resource definition](https://docs.gravitee.io/apim/3.x/apim\_kubernetes\_operator\_user\_guide\_reusable\_resources.html). This enables you to define resources such as cache or authentication providers once only and maintain them in a single place, and then reuse them in multiple APIs - any further updates to such a resource will be automatically propagated to all APIs containing a reference to that resource.

In future releases, the GKO will support additional functionality to enable the following:

* Using the GKO as an Ingress Controller for deploying Ingresses to an API Gateway.
* Deploying Gravitee products (API Management, Access Management, Alert Engine).
* Improving automation processes by covering CICD aspects when using Kubernetes with APIM.
* Managing most API Management resources without directly relying on the Console or on the Management API.

In this guide, you can learn how to work with custom resource definitions (CRDs) and how to synchronize your API CRDs with an existing management API, including how to start, stop, update, and delete your APIs - all this with examples.

The Gravitee Kubernetes Operator comes with three CRDs - `ManagementContext`, `ApiDefinition`, and `ApiResource`. They are described in detail further in this section.