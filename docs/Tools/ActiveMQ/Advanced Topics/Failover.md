This page will detail ActiveMQ's failover functionality.

The Failover transport layers reconnect logic on top of any of the other transports. **The configuration syntax** allows you to specify any number of composite URIs.

The Failover transport randomly chooses one of the composite URIs and attempts to establish a connection to it.

If it does not succeed, or if it subsequently fails, a new connection is established choosing one of the other URIs randomly from the list.

Configuration Syntax:

 `failover:(uri1,...,uriN)?transportOptions&nestedURIOptions`

or

 `failover:uri1,...,uriN`

Example:

`failover:(tcp://localhost:61616,tcp://remotehost:61616)?initialReconnectDelay=100`

Transport Options we use :

| Option Name   | Default Value | Description                                                                    |
| ------------- | ------------- | ------------------------------------------------------------------------------ |
| backup        | false         | Initialize and hold a second transport connection - to enable fast failover.   |
| randomize     | true          | If true, choose a URI at random from the list to use for reconnect.            |
| trackMessages | false         | Keep a cache of in-flight messages that will flushed to a broker on reconnect. |

For more options refer to : [The Failover Transport](https://activemq.apache.org/failover-transport-reference)

## Using Randomize

The Failover transport chooses a URI at random by default. This effectively load-balances clients over multiple brokers. However, to have a client connect to a primary first and only connect to a secondary backup broker when the primary is unavailable, set 'randomize=false'.

Example:

`failover:(tcp://primary:61616,tcp://secondary:61616)?randomize=false`

Notes

	Under the Failover transport send operations will, by default, block indefinitely when the broker becomes unavailable. There are two options available for handling this scenario. First, either set a [TransportListener](http://activemq.apache.org/maven/apidocs/org/apache/activemq/transport/TransportListener.html) directly on the [ActiveMQConnectionFactory](http://activemq.apache.org/maven/apidocs/org/apache/activemq/ActiveMQConnectionFactory.html), so that it is in place before any request that may require a network hop or second, set the 'timeout' option. The 'timeout' option causes the current send operation to fail after the specified timeout.

Example:

`failover:(tcp://primary:61616)?timeout=3000`

In this example if the connection isn’t established the send operation will timeout after 3 seconds. It is important to note that the connection is not killed when a timeout occurs. It is possible, therefore, to resend the affected message(s) later using the same connection once a broker becomes available.

### Transactions

The Failover transport tracks transactions by default. In-flight transactions are replayed upon re-connection. For simple scenarios this works as expected. However, there is an assumption regarding acknowledged (or consumer) transactions in that the previously received messages will automatically be replayed upon re-connection. This, however, is not always true when there are many connections and consumers, as re-delivery order is not guaranteed as stale outstanding acknowledgements can interfere with newly delivered messages. This can lead to unacknowledged messages.

Re-delivery order is tracked and a transaction will fail to commit if outstanding messages are not redelivered after failover. A javax.jms.`TransactionRolledBackException` is thrown if the commit fails. In doubt transactions will result in a rollback such that they can be replayed by the application. In doubt transactions occur when failover happens when a commit message is in-flight. It is not possible to know the exact point of failure. Did failure happen because the transaction commit message was not delivered or was the commit reply lost? In either case, it becomes necessary to rollback the transaction so that the application can get an indication of the failure and deal with any potential problem.

## Priority Backup

If brokers are available in both local and remote networks, it's possible to specify a preference for local brokers over remote brokers using the priorityBackup and priorityURIs` options.

Consider the following URL:

`failover:(tcp://local:61616,tcp://remote:61616)?randomize=false&priorityBackup=true`

Given this URL a client will try to connect and stay connected to the local broker. If local broker fails, it will of course fail over to remote. However, as priorityBackup parameter is used, the client will constantly try to reconnect to local. Once the client can do so, the client will re-connect to it without any need for manual intervention.

By default, only the first URI in the list is considered prioritized (local). In most cases this will suffice. However, in some cases it may be necessary to have multiple “local” URIs. The priorityURIs option can be used to specify which URIs are considered prioritized.

Example:

`failover:(tcp://local1:61616,tcp://local2:61616,tcp://remote:61616)?randomize=false&priorityBackup=true&priorityURIs=tcp://local1:61616,tcp://local2:61616
`
In this case the client will prioritize either local1 or local2 brokers and (re-)connect to them if they are available.

## Configuring Nested URI Options.

Common URI options can be configured by appending them to the query string of the failover URI where each common URI option has the prefix: nested.` 

Example - instead of doing this:

`failover:(tcp://broker1:61616?wireFormat.maxInactivityDuration=1000,tcp://broker2:61616?wireFormat.maxInactivityDuration=1000,tcp://broker3:61616?wireFormat.maxInactivityDuration=1000)`

do this:

`failover:(tcp://broker1:61616,tcp://broker2:61616,tcp://broker3:61616)?nested.wireFormat.maxInactivityDuration=1000`

Any option that can applied to the query string of an individual URI is a candidate for use with the nested option.

## Option Precedence

If the same option is specified as both an individual URI option and as a nested option, the nested option definition will take precedence.