## Root Cause

Proxy Configuration in the "ibi-daas" services that are deployed in Kubernetes caused the issue because OpenTelemetry requires HTTP 2.0 support in order to work and the proxy servers on Intel do not support HTTP 2.0.

So, when the service is trying to report to OpenTelemetry through the Proxy servers it doesn't work.


## Initial thoughts

At the beginning there were some assumptions that may still be relevant to other issues in the future:

1. OpenTelemetry cannot track requests forwarded by ocelot from the gateway service.
2. Kubernetes Service doesn't allow the OpenTelemetry traffic.
3. Kubernetes is blocking the OpenTelemetry traffic.

All of these theories (except the 1st one were rejected based on the infra diagram below).

## OpenTelemetry Infra

![](iBI%20DaaS%20OpenTelemetry%20Issue.svg)


## Details

At the beginning we thought that the problem is with the Kubernetes cluster.
When we used the Development API Gateway [iBI API Gateway Dev](https://ibi-daas-api-dev.intel.com/swagger/index.html?urls.primaryName=hello%20-%20v1) to execute endpoints in the Hello service (e.g. `GET /hello/user` `POST /hello/user`) no traces or logs were reported to Elastic.

But when we executed endpoints directly from the Hello service (e.g. `GET /hello/user` `POST /hello/user`) using kubectl port forwarding both logs and traces were reported using OpenTelemetry successfully.

In addition, when we ran the Hello docker on our local machines (without the gateway as middleman) it works as well.

## Solution

The solution is to exclude the domain name of the Kubernetes cluster using the NO_PROXY environment variable.

Example Configuration:

```yaml
ibi:  
  namespace: daas-dev  
  name: ibi-daas-databricks-dev  
  image: ger-is-registry.caas.intel.com/pda/ibi-daas/ibi-daas-databricks:0.11.76  
  replicaCount: 2  
  env:  
  - name: ASPNETCORE_ENVIRONMENT  
    value: Staging  
  - name: NO_PROXY  
    value: ".svc.cluster.local"  
  - name: HTTP_PROXY  
    value: "http://proxy-chain.intel.com:911"  
  - name: HTTPS_PROXY  
    value: "http://proxy-chain.intel.com:912"  
  - name: OpenTelemetrySettings__ExporterOptions__Endpoint  
    value: "http://otel-receiver-collector-service.open-telemetry.svc.cluster.local:4317"  
  service:  
    enabled: true  
    defaultPort: 80  
  ingress:  
    enabled: false

mssql:  
  enabled: true
```

Here, we can see that the HTTP_PROXY and HTTPS_PROXY environment variables are defined.

Without the NO_PROXY environment variable (which excludes `.svc.cluster.local` in this case, which is the suffix of the kubernetes cluster services) OpenTelemetry wouldn't work.

But since `NO_PROXY` is defined to exclude `.svc.cluster.local` and in the OpenTelemetry connection string is `http://otel-receiver-collector-service.open-telemetry.svc.cluster.local:4317` (which ends in `.svc.cluster.local`) the application will ignore the Proxy servers and will contact the OpenTelemetry receiver (which will now work 😀).