## Simulating Windows Production Load

In our DevOps testing of ActiveMQ the goal was to achieve (and surpass) the volume of clients & throughput on an ActiveMQ broker.

Before reading this document, please refer to the "Windows Environment Analysis" page for more details.
## Testing

Apache JMeter is a "solution of choice" for ActiveMQ performance tests and production load simulations.

**Note:** Multiple functional, performance and failover tests were performed on Testing Environment (**icsl6673**) during last 3 weeks and the cluster performed exceptionally.

**In the below test** "icsl6673 broker" were loaded by two simultaneous JMeter Test Plans running on two powerful machines (**hasibiapp0021**, **hasibiapp0022**).

See performance test results below for reference:

#### ProductionSimulation1.jmx
Test Plan (Left side) and Summary Report (Right side)

![](Production%20Simulation%201.png)

#### ProductionSimulation2.jmx
Test Plan (Left side) and Summary Report (Right side)

![](Production%20Simulation%202.png)

**Additional test** especially for **large messages flow** was performed independently to check the current setup limitations.

The test took in mind the 1 gigabit bandwidth of the ActiveMQ host (which already sets the maximum throughput to 1 • 125 megabyte messages per second).

We configured 2 server to send 150 Mb messages only without a limit to speed and the average throughput has reached rate of 1 msg/2 sec (75Mb/sec) via Gigabit Ethernet Interface (with the 125Mb/sec theoretical maximum).

## Testing Summary

* Test plan duration: > 142 hours (~6 days)
* Total threads (clients) in 2 main tests:  >10000
    * 1 producer for 150Mb messages with Throughput of 5/min (Which A LOT more than the calculated theoretical of 8.7 messages/hour for 100Mb size )
    * 3 producers for 40Mb messages with Throughput of 12/min (To raise the average message size even more)
    * 250 producers for 40Kb messages with Throughput of 30/sec (To raise the average message size with high throughput)
    * 250 producers for 1Kb messages with Throughput of 200/sec (To simulate minimum message size with high throughput and reduce the average)
    * 1000 producers for 1Kb messages with Throughput of 200/sec (To surpass the average throughput of Oregon which is 40 messages/second)
    * 8500 Total consumers - both active and idle (To simulate the  production active & passive consumer count)
    * Average enqueue rate > 430 messages/sec (Maximum of 20 million messages in a day was mentioned for ActiveMQ in the past by @sergey which calculates to 231 messages/second, so we achieved more)
    * Maximum Message Throughput is 75Mb/sec
    * More than 437 million messaged were processed by broker with 0% errors (437 million messages / 6 days = 72 million messages per day)
    * Few more additional tests such as "iBI Services" and other "JMeter multithread tests for sync. and async. clients" were performed in parallel to ProductionSimulation1.jmx and ProductionSimulation2.jmx.
* Total clients quantity in "Peak" was over 20,000.
* Maximum broker CPU usage did not exceed 40% with Openwire.
* Maximum broker RAM usage did not exceed 52% (33Gb from total 64Gb)
* According to the tests, it seems that Maximum Message Throughput for current setup is limited by the Gigabit Ethernet Network Interface.
* Additional Failover tests (broker restart / clients restart / network connection interruptions / total cluster outage) were performed and clients connected back in all and no data loss.