## How to identify that the issue is with Logstash deployment?

All Logstash deployments are using a consumer group name such `ls_*` in their name, where `ls` indicates Logstash, and the rest is the name of the deployment.

**Examples:**
* ls_eck_arm_libi_k8s_idc                - Indicates Logstash for arm
* ls_eck_alerts_execution_libi_k8s_idc   - Indicates Logstash for alerts execution
* ls_eck_cmgr_libi_k8s_idc               - Indicates Logstash for cmgr
* ls_eck_health_libi_k8s_idc             - Indicates Logstash for health
* ls_eck_daas_api_gateway_libi_k8s_idc   - Indicates Logstash for daas api gateway
* ls_eck_etl_libi_k8s_idc                - Indicates Logstash for etl
* ls_eck_errors_libi_k8s_idc             - Indicates Logstash for errors

**Example Ticket:** `Kafka Consumer ls_eck_arm_libi_k8s_idc has data issue on site iil`

We can see the consumer group begins with ls_*, therefore it's a Logstash deployment.

## How to fix this:
For the Lag to be fixed, we need to restart the problematic Logstash deployment:

1. Take the name of the Logstash deployment (e.g. `ls_eck_arm_libi_k8s_idc`).

2. Check which site does this belong to ("idc" for Israel, "or" of America).

2. Open Rancher [ger-pda-cluster](https://ger.caas.intel.com/c/c-xx6ck/) and login.
3. **Navigate to the "daas" project in ger-pda-cluster (1,2,3)** ([daas project](https://ger.caas.intel.com/p/c-xx6ck:p-grj9r/))
![](Consumer%20Lag%20BKM%20-%20Logstash%201.jpeg)
1. **Expand "Resources" and navigate to "workloads" (1)**.
![](Consumer%20Lag%20BKM%20-%20Logstash%202.jpeg)
1. In the **workloads page (1)** use the **search bar (2)** to search for **"logstash-prod" (3)** .<br>
**logstash-prod is the name of the namespace, not the Logstash deployment!!**. <br>
You should then see all the **logstashes we have deployed (4)**.
![](Consumer%20Lag%20BKM%20-%20Logstash%203.jpeg)
1. Look for the one that has a similar name to the problematic one from the ticket.<br>
For Example:<br>
`ls_eck_arm_libi_k8s_idc` will be `ibi-arm-prod-logstash`.<br>
`ls_eck_alerts_execution_libi_k8s_idc` will be `ibi-alerts-execution-prod-logstash`.<br>
1. When you find **it (1)**, click on the **3 dots next to it (2)**, and then click on **"Redeploy" (3)**.
![](Consumer%20Lag%20BKM%20-%20Logstash%204.jpeg)
1. The problem should be fixed and the ticket should be closed in the next 15 minutes.

If this does not resolve the issue, please contact the OnCall and the DevOps team.