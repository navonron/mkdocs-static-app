## ⚠️⚠️⚠️ THIS METHOD WILL BE DEPRECATED IN RANCHER IN **VERSION 2.7** SO YOU NEED TO MIGRATE TO: [Install iBI Chart using Helm CLI](Install%20iBI%20Chart%20using%20Helm%20CLI.md) ⚠️⚠️⚠️

<br><br>

> ⚠️ **I am not kidding, do not use this method, this method will be deprecated and was used in older version of Rancher and is not supported anymore!!!!** ⚠️

<br><br>

> ⚠️⚠️ **AGAIN UNLESS YOU'RE USING THIS TO MIGRATE FROM THIS OLD VERSION TO THE NEWER HELM CLI VERSION DO NOT CONTINUE AND USE [Install iBI Chart using Helm CLI](Install%20iBI%20Chart%20using%20Helm%20CLI.md)** ⚠️⚠️

<br><br>


This page will show you how to install the iBI Service in Kubernetes using Rancher CLI (A tool developed by the DevOps team) in Azure DevOps Release Pipeline.

Installing Rancher Apps for Rancher version older than 2.5.11 are not supported natively in any CLI tool, so the DevOps team built one that will do it.
## Requirements

In the artifacts add a new Build artifact (required by "Install Helm Chart")

* Project : `DevOps`
* Source (build pipeline) : `Rancher CLI`
* Default Branch : `master`
* Default Version : `Latest from a specific branch with tags`
* Source alias : `_Rancher` (important for the task group "Install Helm Chart")

![](Pasted%20image%2020231024130327.png)

After adding the artifact it should look like this in the release pipeline:

![](Pasted%20image%2020231024103329.png)

## Deployment Stage

Add a new stage named whatever you want ("Production" in this example).

![](Pasted%20image%2020231024103406.png)

Under this stage add a new agent job.
In the display name change this to whatever you want.

In the Agent pool make sure to select `On-prem (Windows)`.

![](Pasted%20image%2020231024103427.png)

## Add Variable Groups & Pipeline Variables

This section is based on the pipeline I observed by iBI and is in no way related to DevOps.
### Pipeline variables

Navigate to the Variables section of the release pipeline and add a new Pipeline Variable named `SERVICE_NAME` with the value of your service name for example (`hello`)


![](Pasted%20image%2020231024102957.png)

### Variables Groups

In variables groups add the "Rancher GER" which contains the `GER_PDA_CLUSTER_NAME` and `  RANCHER_API_URL` and `RANCHER_TOKEN` that are later used in the "Install Helm Chart" to authenticate with Rancher rest API.

![](Pasted%20image%2020231024103015.png)

## Add tasks

1. In the Agent Job Add the "Install Helm Chart" task group and fill in the variables like below
![](Pasted%20image%2020231024103601.png)

2. Configure "Install Helm Chart"
	* Display name: `whatever you want`
	* APP_NAME: `ibi-daas-$(SERVICE_NAME)-prod` (uses the `SERVICE_NAME`)
	* CLUSTER_NAME: `$(GER_PDA_CLUSTER_NAME)` - taken from the Variable Group `Rancher GER`
	* NAMESPACE: `daas` for Production, `daas-dev` for Development & Staging
	* RACHER_API_URL:  `$(RACHER_API_URL)` - taken from the Variable Group `Rancher GER`
	* RANCHER_TOKEN:  `$(RANCHER_TOKEN)` - taken from the Variable Group `Rancher GER`
	* VALUES_PATH: `$(System.DefaultWorkingDirectory)/_ibi-daas-$(SERVICE_NAME)/k8s/values-staging.yml` (uses the `SERVICE_NAME`)

That's it! you can now add your service repository as a build artifact with the values.yml that's needed to deploy the service and the pipeline is ready 😀.