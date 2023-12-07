Monitoring Stale and EOLed components is important for cleaning up the system.

- [ ] Kubernetes Empty Namespaces (All Clusters)
- [ ] Kubernetes endpoints from nginx-ingress-controller
- [ ] Kubernetes Projects (Deprecated by Rancher)
- [ ] Kubernetes unused Service for over X Time no traffic
- [ ] Kubernetes unused jobs that are finished (rclone)
- [ ] Kubernetes services without target workloads
- [ ] Kubernetes ingresses without target workloads
- [ ] Kubernetes ConfigMaps that aren't used
- [ ] HighWire LTM & GTM without Traffic

## Nightly Job (HTML Report) for ALL below

Need to make nightly job that uses Python to check if there are any violations in the below categories, if so, send an email to the DevOps Team with the details of the violations.

## Stale Ingresses

Stale Ingresses are ingress services that are pointing to an endpoint that doesn't actually exist.

In the examples below there are three ingresses that show three ingress objects that are pointing to an ingress-*gibberish* service that doesn't exist (if it does exist, an ingress-*gibberish* service usually doesn't point to anything anyways).

![](Kubernetes%20Cleanup%20Routine%201.png)![](Kubernetes%20Cleanup%20Routine%202.png)

## Stale "Ingresses"

These are "Service" objects in Kubernetes, no "Ingress" objects..
These services are not pointing to workloads which actually exists.

These are residual services from a Rancher version 2.5 upgrade to 2.6.

Rancher 2.5 had created "ingress-*gibberish*" services that were actually services (not ingresses) 
For example:

- ingress-91d0593bde73f8f820a98f1986c7b449
- ingress-eeea4e4227dda0206c28391964f8c795
- ingress-0c636586fe428e5ad16c8ac96cbff52b

![](Kubernetes%20Cleanup%20Routine%203.png)

These services are stale and do not lead to anything that actually exists.

These "Fake Ingresses" as I like to call them remained in the cluster after the upgrade. One of these fake ingresses was associated with the service being scaled, they cause pods failures so they must be cleaned from the cluster.
