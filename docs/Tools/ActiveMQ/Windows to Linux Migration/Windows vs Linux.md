# ActiveMQ - Windows vs Linux

## Introduction

This page is dedicated to detailing the differences between the current Windows ActiveMQ cluster and the new Linux ActiveMQ cluster.

Here, we delve into crucial insights gathered from extensive observations.

## Servers Comparison
### Background

ActiveMQ stats were measured to set Linux cluster targets.

The Linux machines have MUCH lower CPU/RAM compared to current Windows machines.

<table>
    <th>#</th>
    <th>Windows Server</th>
    <th>Linux Server</th>
    <tr>
        <td>CPU Physical Cores</td>
        <td>28 Cores</td>
        <td>4 Cores</td>
    </tr>
    <tr>
        <td>CPU Logical Cores</td>
        <td>56 Cores</td>
        <td>-</td>
    </tr>
    <tr>
        <td>RAM</td>
        <td>192GB</td>
        <td>64GB</td>
    </tr>
    <tr>
        <td>Hyper Threading Enabled</td>
        <td>Yes</td>
        <td>No (but can be enabled)</td>
    </tr>
    <tr>
        <td>ActiveMQ Version</td>
        <td>5.9.0</td>
        <td>5.16.2</td>
    </tr>
    <tr>
        <td>Operation System</td>
        <td>Windows Server 2016</td>
        <td>SLES 12</td>
    </tr>
</table>

A Link for direct comparison between the CPUs: [CPU Comparison](https://askgeek.io/en/cpus/vs/Intel_Xeon-Gold-5120-vs-Intel_Xeon-E3-1270-v5)

### Comparison focus

1. Total Consumer count
2. Total Producer count
3. Maximum messages size
4. Average message size
5. Minimum message size
6. Throughput (Total Enqueue count / Uptime) - to indicate network & disk usages + throughput.

### Goals

* Simulate Oregon cluster (Since the Oregon cluster superior in volume) *and try to
* Check if the Linux Super Micro servers are a good fit.

## Clusters Comparison

To further elaborate on the subject we will use the metrics from all environments to make comparisons:

Left: IDC Linux Testing

Middle: IDC Windows Prod

Right: OR Windows Prod

![](Windows%20vs%20Linux%201.png)

According to Apache, there is no theoretical limit to the number of clients ActiveMQ can handle (e.g the more horse power you have the more threads you can run).

But we have limited resources on the Linux servers (CPU bottleneck) so:

We were focusing on achieve the following in our cluster (Oregon was the cluster we wanted to achieve since it handles large amount of clients and traffic): 

- ~15,000 Consumers.
- ~1,600 Producers.
- ~37 messages/seconds throughput.
- ~16,000 Bytes Average message size

As you can see from the results above, we were able to achieve even more than the expected amount of consumers but more producers caused the CPU to go over the recommended limit.

According to Apache &  the ActiveMQ community broker host CPU should not go over 80%.



> ⚠️ Synchronous messaging requires a greater number of threads than asynchronous delivery. Using more threads causes the broker to incur the overhead of context switching, which requires more work from the host's CPU. This could cause a slowdown in the queueing and dispatching of messages, and ultimately could lead to lower message throughput.

> ⚠️ **Threads potential**
> It's important to note that while we recommend ~1,500 max producers and ~6,500 max consumers there are few things to keep in mind:
> 
> - We aimed for higher throughput (messages per second) than the ones in production.
> - We aimed for higher average on messages size.
> - We aimed to achieve the producers amount of the biggest cluster (Oregon).
> - The consumer amount of Oregon cannot be achieved in our Linux servers.
>
> 
> We want to ensure that the cluster is future resistant to changes and thus don't want to put the cluster to it's limits.
> 
> 
> - The throughput for all the tests was set to be identical (~170 messages / sec)
> - The average message size was 160kb.
> - The max message size was 100mb.
> 
> But still we stick with the recommendation above.

### CPU Limit:
According to Apache's recommendation, CPU usage should be less than 80% on the broker host.

#### The CPU usage on ActiveMQ:

- In Oregon Windows environment is: **~0.3%**
- In IDC Windows environment is: **~0.2%**
- In IDC Linux environment (with greater load than IDC): **~0.1%**

From the tests above, we tested the Linux servers with more than the load of the production (in messages per second & messages size) and the conclusion is:

1. The throughput of Messages/Second in the Linux server testing is higher than IDC & PDX combined.
2. The average messages size is larger than both production combined.
3. The Maximum message size is larger than both sites (and from our estimations we also sent a larger amount (5 per minute) of "Maximum messages" than production in average, explained below)

![](Windows%20vs%20Linux%202.png)

> ⚠️ It's important to note that even though the Linux environment server has 4 logical cores against 56 in Windows, it manages to handle the CPU load.

### RAM Limit

The RAM usage is affected by the messages that are sent to ActiveMQ.

It is used and directly affected by broker connections, as a cache for ActiveMQ messages to achieve better performance when messages are consumed fast enough (when lag is low).

### The RAM usage on ActiveMQ

 - In Oregon Windows environment is: **~12GB out of 192GB**
 - In IDC Windows environment is: **~5GB out of 192GB**
 - In IDC Linux environment (Running greater load than IDC & Oregon in throughput of messages): **~7GB out of 64GB**

The Heap size for ActiveMQ is set at 35GB, and the broker is configured to utilize 70% of that capacity, approximately 24.5GB. This allocation is deemed sufficient, taking into account the observed RAM usages.

## Final DevOps recommendations
### Limitations for Super-Micro servers (Linux)

> ⚠️ Producers can be swapped for consumer, but not the other way around.

> ⚠️ These recommendation are based on ActiveMQ average message size, minimum message size, maximum message size and throughput seen in Production cluster.

> ⛔ **Future resistant**
> Even though it's within the limits of recommended and can handle the IDC load for now, we recommend to switch powerful Linux servers ASAP. 

<table>
    <th>Limitation</th>
    <th>Explanation</th>
    <th>DevOps Notes</th>
    <tr>
        <td>
            Up to 15,000 Consumers (Subscribers)
        </td>
        <td>
            Passive Consumers: 20% more than the largest observed number on Production.
            They utilize less CPU than Producers.
        </td>
        <td>
            The broker should be able to handle more than this as consumers do not cause heavy CPU usage.
        </td>
    </tr>
        <td>
            Up to 1,500 Producers (Publishers)
        </td>
        <td>
            In production, the maximum number of Producers observed is 1,500. Producers impose greater stress on the ActiveMQ broker's CPU and RAM compared to Consumers. To align with the recommended CPU limit on a 4 CPU cores server, it is advisable to set the limit at 1,500 Producers.
        </td>
        <td>
            With 1,500 Producers and a load 30% heavier than the current iBI usage, the CPU utilization ranges from approximately 60% to 75%. To avoid potential issues, it is recommended to set this as the limit for a 4-core CPU.
        </td>
    <tr>
    </tr>
        <td>
            TCP Protocol, port 61616
        </td>
        <td>
            Port 61616 is the ActiveMQ port that is configured to be used for communication.
            It is using the TCP Protocol
        </td>
        <td>
            -
        </td>
    <tr>
    </tr>
        <td>
            Set maximum PIDs on the machine: 4194303
        </td>
        <td>
            Set the number of PIDs allow on the host to 4194303.
        </td>
        <td>
            ActiveMQ can create processes up to 33% of the host's process limit.
            Each ActiveMQ client generates a process.
            The initial value was about 32,000. 
            Although 4 million PIDs may seem excessive, the community recommends removing this limit for optimal performance.
        </td>
    <tr>
    </tr>
        <td>
            Set file descriptor limit to 65536
        </td>
        <td>
            Set the limit on the number of files ActiveMQ can write on the host to 65536.
        </td>
        <td>
            ActiveMQ creates a file for each client connection (Producers + Consumers).
            ulimit -n sets the maximum file descriptor plus 1.
            The default value on the Super-Micro machines was 8192 - which means a limit of ~8192 clients.
            Increasing this number allows more clients to be connected to the broker.
        </td>
    <tr>
        <td>
            NFS Tier 1
        </td>
        <td>
            Tier 1 NFS is the fastest NFS tier in Intel.
        </td>
        <td>
            NFS tier 1 is capable of handling the throughput of current Production in all iBI sites combined.
        </td>
    </tr>
    <tr>
        <td>
            NFS Capacity at least 100GB
        </td>
        <td>
            If the NFS won't have the necessary capacity to handle this kind of situations we will have a bigger issue then queue size.
        </td>
        <td>
            100GB is a good starting point since the size of data & log only came up to 25GB when production had issues.
            In usual times the size of NFS data is around ~1GB-2GB.
        </td>
    </tr>
</table>


## References:
**ulimit**<br>
[ulimit (oracle.com)](https://docs.oracle.com/cd/E19683-01/816-0210/6m6nb7md9/index.html)

**PIDs Maximum**<br>
[Change the kernel max PID number (Stack Exchange)](https://unix.stackexchange.com/questions/162104/how-to-change-the-kernel-max-pid-number)<br>
[Linux Increase Process Identifiers Limit (cyberciti.biz)](https://www.cyberciti.biz/tips/howto-linux-increase-pid-limits.html)

**UserTasksMax**<br>
[How to modify systemd DefaultTasksMax / TasksMax / UserTasksMax settings | Support | SUSE](https://www.suse.com/support/kb/doc/?id=000015901)

**Other**<br>
[How to solve java.lang.OutOfMemoryError: unable to create new native thread - Mastertheboss](http://www.mastertheboss.com/jbossas/monitoring/how-to-solve-javalangoutofmemoryerror-unable-to-create-new-native-thread/)<br>
[How to Set Limits for User Running Processes in Linux (tecmint.com)](https://www.tecmint.com/set-limits-on-user-processes-using-ulimit-in-linux/)