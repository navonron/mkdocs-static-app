# 2023-09-05 - Nginx Retry

## Summary

When a response takes more than 1 minute it gets retried a 2nd and 3rd time.
This only happens when a service is hosted in K8s.

## Details

The issue was observed by Alex.

* Direct service call  doesn’t suffer from this.
* Call via gateway  **deployed locally** doesn’t suffer from this as well

According to Alex:

<i>This makes me think that Kubernetes reruns http request on another pod if it doesn’t get response in 1 minute.

What happens next is that first  request  succeeded, second fails (because of the nature on implementation), client gets **error**</i>

### Evidence

On `ibi-daas-gateway-dev-1`

![](2023-09-05%20-%20Nginx%20Retry%201.png)

1 minute after on `ibi-daas-gateway-dev-0`

![](2023-09-05%20-%20Nginx%20Retry%202.png)

Service logs in Elastic

![](2023-09-05%20-%20Nginx%20Retry%203.png)

## Affected Services

All services are affected by this as this is an nginx default to allow this behavior.

The behavior needs to fixed only in:

* ibi-daas-gateway
* ibi-daas-login
## Resolution

> ⚠️ Ocelot on the iBI gateway services is configured to return timeout after 90 seconds to 3600 is obviously enough.
> 
> ![](Pasted%20image%2020230918172404.png)

Add this below **annotations** to the **Annotations of the ingress yaml** of the service.

```yaml
kubernetes.io/ingress.class: nginx
nginx.ingress.kubernetes.io/proxy-connect-timeout: '3600'
nginx.ingress.kubernetes.io/proxy-read-timeout: '3600'
nginx.ingress.kubernetes.io/proxy-send-timeout: '3600'
```

This will increase time ingress timeout to 1 hour instead of 1 minute.
Then, Ocelot will be the one to timeout the user according to the iBI needs and configurations (currently set to 90 seconds as mentioned above).

### Annotations Breakdown

**proxy-connect-timeout** - Sets the timeout for establishing a connection with a proxied server. It should be noted that this timeout cannot usually exceed 75 seconds.

**proxy-read-timeout** - Sets the timeout in seconds for reading a response from the proxied server. The timeout is set only between two successive read operations, not for the transmission of the whole response.

**proxy-send-timeout** - Sets the timeout in seconds for transmitting a request to the proxied server. The timeout is set only between two successive write operations, not for the transmission of the whole request.

## Conclusion

The issue was resolve by increasing the timeout of the requests (which is what triggers the retry on a different container).
