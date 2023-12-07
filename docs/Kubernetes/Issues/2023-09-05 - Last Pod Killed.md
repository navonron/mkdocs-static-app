# 2023-09-05 - Last Pod Killed
## **Summary**

A workload (StatefulSet / Deployment) in the cluster is constantly killing off the container in the place "count -1".

Examples:

- A StatefulSet with replicas set to 3, the 3rd container will be killed (1 & 2 will stay up).
- A StatefulSet with replicas set to 2, the 2nd container will be killed (1 will stay up).
- A StatefulSet with replicas set to 1, the 1rd (and only) container will be killed (The application will not work).

## **Details**

The `ibi-daas-catalog-prod` service has had issues with spawning the "last" container.
In a first glance in seems like a Kubernetes pod scaling issue, but it's a Rancher issue.

I do not know if it's a bug or not and I didn't find anything on the internet to suggest otherwise..

The issue started when the `ibi-daas-catalog-prod` service was upgraded using Azure DevOps release, and the Helm chart version was changed from version 0.2.4 to 0.3.0 .

This upgrade replaces the discovery mechanism that is EOLed by Rancher in Rancher 2.6.

### **Issues**

- Pods in a Kubernetes cluster were experiencing repeated scaling issues where one pod failed during scaling operations, leading to an unexpected pod count.

### **Affected Services**

- ibi-daas-catalog-prod
- ibi-daas-query-dev
### **Investigation Steps**

1. I tried upgrading the helm chart using the Azure DevOps release of the catalog service.
2. I tried recreating the service manually from the Rancher UI (thought it might change something for Rancher).
## Root Cause

The issue was finally identified as a residual problem from a Rancher version 2.5 upgrade to 2.6. Rancher 2.5 had created "ingress-*gibberish*" services that were actually services (not ingresses) 
For example:

- ingress-91d0593bde73f8f820a98f1986c7b449
- ingress-eeea4e4227dda0206c28391964f8c795
- ingress-0c636586fe428e5ad16c8ac96cbff52b

![](2023-09-05%201.png)

These services are stale and do not lead to anything that actually exists.

These "Fake Ingresses" as I like to call them remained in the cluster after the upgrade. One of these fake ingresses was associated with the service being scaled, causing the pod count - 1 issue.

## Resolution

To resolve the issue, the following steps were taken: 

1. **Identification:** The presence of the fake "ingress-*gibberish*" services created by Rancher 2.5 was identified as the root cause.
2. **Service Cleanup:** The fake ingress services were deleted from the Kubernetes cluster to eliminate the interference with the scaling operation.
	1. **Scaling Success:** With the removal of the problematic service, scaling operations for the affected service proceeded without any issues, and the desired pod count was achieved.
## Conclusion

The scaling problem was successfully resolved by identifying and removing the unexpected "ingress-*gibberish*" services that were created by Rancher 2.5 but remained in the cluster after the upgrade to Rancher 2.6. This cleanup action allowed the desired scaling behavior to be achieved without one pod failing during scaling operations.
