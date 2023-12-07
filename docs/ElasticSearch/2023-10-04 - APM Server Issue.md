# 2023-10-04 - APM Server Issue

## Problem as reported by ????

Open Telemetry metrics are not getting to Elastic and cannot be viewed.

I went to Kubernetes were OpenTelemetry was hosted

The receiver OTPL container was healthy and were no errors.
The exporter (the one pushing data the Elastic APM) show an error that it cannot export data because the server is unreachable.

I checked Kafka topics and messages were going in but not going out (like observed in OTLP containers).

I went to Elastic cloud console and APM was unhealthy and down.

I asked Roy Prigat to force restart it because I didn't have permissions

Restart solved the issue.
## APM Server was unreachable



## Elastic Support Agent Response


*When I checked your APM server, I noticed that the APM was not reachable on 4 October:*

```
Oct 4, 2023 @ 09:53:33
Could not communicate with fleet-server Checking API will retry, error: fail to checkin to fleet-server: Post "https://instance-0000000017:8220/api/fleet/agents/94908f57-1cea-4691-afa9-f30ee22e95c3/checkin?": context canceled
```

*Elastic Agent was then stopped and the server was exited:*
```
Oct 4, 2023 @ 09:53:33
Shutting down Elastic Agent and sending last events...
+++
Oct 4, 2023 @ 09:54:05.254
fleet-server
+++
Oct 4, 2023 @ 09:54:05.255
Starting stats endpoint
+++
Oct 4, 2023 @ 09:54:08.747
Fleet Server exited
```

*The server seemed to have stopped because it was overloaded*

![](Pasted%20image%2020231023151701.png)

![](Pasted%20image%2020231023151720.png)

*The node then repeatedly tried to restart automatically but failed.*

```
Oct 4, 2023 @ 09:54:2
Boot fleet-server
+++
Oct 4, 2023 @ 09:54:20.620
starting communication connection back to Elastic Agent
+++
Oct 4, 2023 @ 09:54:20.669
waiting for Elastic Agent to send initial configuration
+++
Oct 4, 2023 @ 09:54:21.030
Fleet Server exited
```

*The forced restart you performed permitted to bring the node up again.

Regards,  
Eyapènè*

*Access Your Case: [https://support.elastic.co/cases/5008X00002UmWCHQA3](https://support.elastic.co/cases/5008X00002UmWCHQA3)*
