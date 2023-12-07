ActiveMQ's **InactivityMonitor** is an active thread that closes inactive and incorrectly functioning connections.

### Connection's monitoring components:

1. **"maxInactivityDuration"** - ActiveMQ may close a connection if no data has been received (default timeout is 30 sec).
2. **"KeepAliveInfo"** (Heartbeats) - message that should be sent on an idle connection in specific time period (default is 30 sec) to prevent it from timing out.

![](Inactivity%20Monitor%201.png)

### Consumer Modes:
1. **Synchronous** - message consumer must call a **"receive()"** method to explicitly fetch a message from a destination (it blocks the channel until a message is received or connection is closed).
2. **Asynchronous** - message listener is registered with a message consumer. When messages for consumer arrive, they are delivered by calling the listener’s **"onMessage()"** function which is invoked from a different thread and hence does not block the channel.

### Synchronous vs Asynchronous message consumer:
Both types of clients were tested using JMeter (alongside active iBI services) using failover transport and **InactivityMonitor** enabled.

JMeter (our testing platform of choice we used to simulate production) JMX Subscriber mode can be set like that:

![](Inactivity%20Monitor%202.png)

### DevOps testing results :
* Each connection has two Inactivity Monitors associated (One in Broker side and on the Client side) and it can be enabled / disabled from either side.
* If Inactivity duration is too long and no Heartbeats were sent on a connection, the **InactivityMonitor** close it. 
* Inactivity Monitor can be disabled from either side (client/broker) to prevent connections from timing out.
* Inactivity Monitor doesn't close Synchronous Consumers
* Inactivity Monitor doesn't close Producers (producers are always active).
* Inactivity Monitor closes all "dead" connections (whenever clients were disconnected graсefully or forcibly)
* Inactivity Monitor periodically closes Asynchronous Consumers and they further reconnect using failover (if configured by the user)
* All clients successfully reconnect to broker after ActiveMQ "outage" and Broker failover tests.

### Broker logs for inactivity monitor action:

![](Inactivity%20Monitor%203.png)

### Client logs for inactivity monitor action:

![](Inactivity%20Monitor%204.png)
