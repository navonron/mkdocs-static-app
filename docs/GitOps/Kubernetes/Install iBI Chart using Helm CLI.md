This page will show you how to install the iBI Service in Kubernetes using Helm CLI in Azure DevOps Release Pipeline.

Some of the tasks (like installing helm & kubectl) are natively supported in Azure DevOps but the task group "Install Helm & Kubectl" has scripts that are supported with the Azure DevOps Agent and will be easier to migrate to another CI CD tool in case Azure DevOps is EOL or not the solution of choice anymore.

## Requirements

In the artifacts add a new Git artifact (required by "Install Helm & Kubectl")

* Project : `DevOps`
* Source (Repository) : `scripts`
* Default Branch : `main`
* Default Version : `Latest from the default branch`
* Source alias : `_scripts` (important for the task group "Install Helm & Kubectl")

![](Pasted%20image%2020231024111611.png)

After adding the artifact it should look like this in the release pipeline:

![](Pasted%20image%2020231024102519.png)

## Deployment Stage

Add a new stage named whatever you want ("Development" in this example).

![](Pasted%20image%2020231024102528.png)

Under this stage add a new agent job.
In the display name change this to whatever you want.

In the Agent pool make sure to select `On-prem (CaaS GER)`.

![](Pasted%20image%2020231024102546.png)

## Add Variable Groups & Pipeline Variables

This section is based on the pipeline I observed by iBI and is in no way related to DevOps.
### Pipeline variables

Navigate to the Variables section of the release pipeline and add a new Pipeline Variable named `SERVICE_NAME` with the value of your service name for example (`hello`)


![](Pasted%20image%2020231024102957.png)

### Variables Groups

In variables groups add the "DesignML Credentials" which contains the `DESIGNML_PASSWORD` and `  DESIGNML_USERNAME` that are later used in the "Install Helm & Kubectl" to authenticate with the harbor helm registry.


![](Pasted%20image%2020231024103015.png)

## Add tasks

1. In the Agent Job Add the "Install Helm & Kubectl" task group and fill in the variables like below
![](Pasted%20image%2020231024102554.png)
2. In the Agent add "Package and deploy Helm charts"

	![](Pasted%20image%2020231024112654.png)

3. Choose Kubernetes Service connection
   * GER (designml) - for GER Production Cluster
   * GER PRE (designml) - for GER Pre Production Cluster
   * AMR (designml) - for AMR Production Cluster
	![](Pasted%20image%2020231024102906.png)


4. Configure "Package and deploy Helm charts"
	* Display name: `whatever you want`
	* Connection Type: `Kubernetes Service Connection`
	* Kubernetes Service Connection: Use the one from step 3
	* Namespace: `daas` for Production, `daas-dev` for Development & Staging
	* Command: `upgrade`
	* Chart Type: `Name`
	* Chart Name: `harbor/ibi-service`
	* Version: `0.3.0`
	* Release Name: `ibi-daas-$(SERVICE_NAME)-dev` (uses the `SERVICE_NAME`)
	* Values File: `$(System.DefaultWorkingDirectory)/_ibi-daas-$(SERVICE_NAME)/k8s/values-staging.yml` (uses the `SERVICE_NAME`)
	* Arguments: `--atomic --create-namespace --debug --install --insecure-tls-verify`
![](Pasted%20image%2020231024102747.png)
![](Pasted%20image%2020231024102639.png)

That's it! you can now add your service repository as a build artifact with the values.yml that's needed to deploy the service and the pipeline is ready 😀.