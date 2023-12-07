This analysis was performed to estimate a size of messages processed by ActiveMQ in the Windows environment.

The analysis was done using a custom Python script that was build by the DevOps team (now replaced with Hawtio Reader).


## General notes

1. The PDX cluster is superior in terms of workloads to IDC.
2. The average message size for all queues in PDX is ~16Kb
3. Only 2 queues in the cluster periodically receive large messages
4. Max message size in other queues in PDX tops at 11.5Kb
5. Min message size in queues is 1Kb
6. `DAAS.QUEUE.CLIENT_MESSAGE.CONSUMPTION` has messages up to 5.7Mb in size.
7. `DAAS.QUEUE.CLIENT_MESSAGE.LOG` has messages up to 100Mb in size.

## Estimating large message throughput (enqueue rate)

![](Windows%20Environment%20Analysis%201.png)

To calculate the throughput of large messages in LOG and CONSUMPTION queues we used the metrics from ActiveMQ:

To calculate the maximum number of large messages we assumed that all of the messages in the queue are the size on the minimum (which makes the average messages size the same as the minimum).

Now we checked how many "maximum messages size" messages needs to be added to the queue to raise the "average message size" back to the original value reported by ActiveMQ.

![](Windows%20Environment%20Analysis%202.png)
![](Windows%20Environment%20Analysis%203.png)

### LOG calculation

When applying the assumptions above and inputting the number to the formula we get:

![](Windows%20Environment%20Analysis%204.png)

For the LOG queue ~23800 in the  variable is the number needed to get the average message back to 16763.

From this we can assume the "worst" maximum message size scenario is 1 message out of 14,574 messages is a 100Mb messages and the enqueue rate is 8.7 messages/hour.

### CONSUMPTION calculation

When applying the assumptions above and inputting the number to the formula we get:

![](Windows%20Environment%20Analysis%205.png)

For the CONSUMPTION queue ~7000 in the  variable is the number needed to get the average message back to 12090.

From this we can assume the "worst" maximum message size scenario is 1 message out of 2,336 messages is a 5.7Mb messages and the enqueue rate is 2.56 messages/hour.

The ActiveMQ Production simulation was built with those statistics in mind.
See JMeter tool, Production simulation and [ActiveMQ Limitations](https://wiki.ith.intel.com/display/pdaopx/ActiveMQ+Limitations) pages in Wiki  for additional details. 