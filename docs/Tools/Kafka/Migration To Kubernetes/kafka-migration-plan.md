# Kafka Migration Plan

**Kafka on K8s IIL LTM:** libi-kafka.iil.intel.com
**Kafka on K8s PDX LTM:** libi-kafka.pdx.intel.com

## **Phase 1**
First deployment week

1. Create LTM Pool for new cluster (on K8s), the LTM port should be 9092 which will balance K8s Kafka port - **DevOps Team**
2. Create DNS A Record for K8s cluster LTM (IIL: libi-kafka.iil.intel.com, PDX: libi-kafka.pdx.intel.com). - **DevOps Team**
3. Yammer Communications to customers  - intermittent failures for 1 hour  - **DevOps Team**
4. Logstash:
    a. Create new logstash config file for the new cluster on k8s (conf file in the logstash repo) - do this for ALL logstashes - **Yehonatan's Team**
    b. Update all logstash workload to add new config (to prevent data loss by reading from old and new cluster). - **Yehonatan's Team**
5. Changes to Global load balancer (ibi-kafka.ibicloud.intel.com:9092):
    a. Add new LTM Pool for new cluster (IIL: libi-kafka.iil.intel.com, PDX: libi-kafka.pdx.intel.com). - **DevOps Team**
    b. Disable old cluster OnPrem brokers in the global load balancer (which will stop new connections to it). - **DevOps Team**
    c. Restart ALL iBI services - **Backend Team**
    d. Restart ALL metricbeat services - **Backend Team** (All except Linux AMQ) + **DevOps Team** (ActiveMQ)
    e. Restart ALL filebeat services - **Backend Team**
6. ARM data should flow to the new cluster along with filebeat and metricbeat.
7. 7 days monitoring  - **DevOps Team**

## **Phase 2**
Week 2 after 7 days of validation

1. Yammer Communications to customers  - intermittent failures for 1 hour  - **DevOps Team**
2. Calendar for phase - **DevOps Team**
3.  Add new K8s cluster at site as read and write in iBI services configs - **Backend Team**
4.  Restart to all services located at Israel to add read and write to the new cluster - **Backend Team**
    a. Update new config only with the new Kafka
    b. Create Wiki how DaaS interacts with Kafka
5.  7 days monitoring  - **DevOps Team**

Restart ALL iBI services - **Backend Team**

## **Phase 3**
Week 3 after 7 days of validation:

1. Yammer Communications to customers  - intermittent failures for 1 hour  - **DevOps Team**
2.  Calendar for phase - **DevOps Team**
3.  Restart to remove the old Kafka cluster (of IDC) from read-only, pending UI team rewire. - after a week of validation - **Backend Team**
4.  Remove config of old Kafka lag from Israel. - **Backend Team**
5.  Loop:
    a. Check which services (both iBI services and services such as metricbeat, filebeat, ActiveMQ, etc.) still report of the old cluster.  - **DevOps Team + Backend Team**
    b. Fix problematic services (restart to apply new config / whatever the case may be).  - **DevOps Team + Backend Team**
    c. Check if Read and Write of messages goes to 0.  - **DevOps Team**
6.  Remove old Kafka configs from ALL Logstash - **Yehonatan's Team**
7.  7 days monitoring  - **DevOps Team**