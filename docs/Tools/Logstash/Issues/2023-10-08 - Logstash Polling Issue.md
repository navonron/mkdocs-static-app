# 2023-10-08 - Logstash Polling Issue

A Consumer needs to be able to consume max.poll.records within max.poll.interval.ms.

the default config is max 500 records in 5 minutes, meaning 0.6 seconds to consume each record.

Bottom line, the Logstash consumers may be too slow, and fail to poll Kafka in time, so they become inactive

The logic is rather simple in the Logstashes, and they were working fine until last week, so something tells me there is latency elsewhere.

Error Message (this is the same error across different Logstashes):

```
[2023-10-04T03:09:03,612][INFO ][org.apache.kafka.clients.consumer.internals.AbstractCoordinator][main][95cbe49cc9b4418e31b20efa9daf5f0207ca7b4294c0cc8ef522f1a0103a09cf] [Consumer clientId=ls_eck_arm-0, groupId=ls_eck_arm_libi_k8s_pdx] Member ls_eck_arm-0-75a685e8-7c10-4654-9371-e6f204bdf9e7 sending LeaveGroup request to coordinator 10.38.168.19:31517 (id: 2147483645 rack: null) due to consumer poll timeout has expired. This means the time between subsequent calls to poll() was longer than the configured max.poll.interval.ms, which typically implies that the poll loop is spending too much time processing messages. You can address this either by increasing max.poll.interval.ms or by reducing the maximum size of batches returned in poll() with max.poll.records.
```