This article will explain in detail the concept of Azure DevOps Runners came to life, and the basic logic behind it (separate article will be written on how to implement it).

## Overview
### Problem Statement

The Azure DevOps Runner project solves the issue of having On Premise (or Cloud) pool of agents that are are platform agnostic, configurable, scalable, and fresh for each job, that are also capable of accessing private instances in the Azure cloud in a private subscription guarded by a Private Link ([What is Azure Private Link? | Microsoft Docs](https://docs.microsoft.com/en-us/azure/private-link/private-link-overview)).

At the time of writing, Microsoft had an offering of “agents on demand” capability in Azure DevOps, which in short - is an option to run Azure DevOps jobs on cloud agents per the user’s needs (spawned an killed automatically).

But because of some weird reason, when a cloud instance is guarded by a Private Link , without enabling the ability of public access (accessing cloud resources from the internet), deployments to the cloud must be executed from Azure DevOps Agents that are on the same network of the private link (either in the cloud or from the Internal network).

### Proposed Work
A Dockerized Azure DevOps Agent, that is platform agnostic, offering the same capabilities of Azure DevOps Cloud Agents (Clean agent environment) and with the ability to access both Cloud Subscriptions resources AND On Premise resources at the same time

(Other HUGE benefits are mentioned in the “Success Criteria” ).

with Kubernetes (Can also be run as a docker container), gives a scalable, and fresh environment for each job on Azure DevOps, alongside the option to access both Cloud Subscriptions resources AND On Premise resources at the same time.

### The End result
Our end result is having a pool of Dockerized Agents running on a Kubernetes cluster in an On Premise network (or in an AKS cluster in the same subscription as the resource you’re trying to upgrade in Azure Cloud).

By using docker containers we can have a preconfigured , configurable & containerized (imaged) environment for each of our needs (docker images with Azure Agent + other custom components).

In the Azure DevOps Agent installation script there are two parameters that are critical for this solution to work properly and are used in the Azure DevOps Agent config:

`--replace` flag in the [config.sh](http://config.sh/) script which will cause a registering agent to replace the previous agent if it has the same name (which will be the case in kubernetes container with the same name).
`--once` flag in the [run-docker.sh](http://run-docker.sh/) script which will cause an agent to be unregistered after completing a job (which will kill the container after a job is done, which makes Kubernetes spawn a fresh agent).


This combination, when used with a kubernetes StatefulSet will cause the following cycle:

![](Overview%20and%20details%201.png)



Each Agent StatefulSet must first be deployed manually by someone or by an existing agent with access to the kubernetes cluster.

It is recommended to use a secret in Kubernetes for the needed parameters (Azure DevOps PAT among them).

Each StatefulSet will create the desired amount of containers (3 in the example above, each container will be an image of an Azure Agent).

After creation, each agent waits for a job for unlimited amount of time (and if an agent with that name existed before in Azure DevOps, will replace it thanks to the `--replace` flag).

After getting a job, the agent will perform all job’s steps and then exit thanks to the `--once` flag.

By exiting the agent, the container will be killed.

The termination of the container in a Kubernetes StatefulSet will cause Kubernetes to recreate the container again, with the same name, but with a fresh image (since kubernetes wants to meet the desired amount of replicas in the StatefulSet) resulting in a clean agent every time.

## Success Criteria
Once the Agent Pool is correctly configured in your network you will all the following benefits (Comparison to Microsoft’s service)

|Feature|Static Self hosted agent|Other solutions|Microsoft’s Cloud Agents|Azure DevOps Runner|
|---|---|---|---|---|
|Clean Agent for each job|No|Yes|Yes|Yes|
|Configurable (Allows Preinstalled components)|Kind of|Kind of|No|Yes|
|Requires Pipeline modifications|No|Yes|No|No|
|Linux based option|Yes|Yes|Yes|Yes|
|Windows based option|Yes|No|Yes|TBD|
|Scalable|No|Yes|No|Yes|
|Shell option on agent|Yes|No|No|Yes|
|Fixed naming convention|Yes|Yes|No|Yes|
|Access to Private cloud, public cloud and On Premise at the same time|Yes|Yes|No|Yes|
|Run Docker jobs|Yes (Will use host docker.sock)|No|Yes|Yes (Will use host docker.sock)|
|Can run on AKS|No|Yes|No|Yes|


## Scope
### Requirements
What requirements should this project fulfill?

Kubernetes cluster (On Premise / AKS within the same subscription as the resource).
Docker registry (Private / Public DockerHub)
Docker registry account with no limits.

### Future Work

List requirements that we know we want to do, but will do later.

Add support for Docker container jobs (run docker inside the dockerized agent).
Release Pipelines to build and deploy images of Azure DevOps Agents automatically.

### Related Documents

Include any links to relevant external documents.

The Dockerized agent was inspired from Microsoft’s documentation.
Run a self-hosted agent in Docker - Azure Pipelines

## Alternatives Considered

[https://github.com/cloudoven/azdo-k8s-agents](https://github.com/cloudoven/azdo-k8s-agents)

[Azure DevOps Self hosted Agents on Kubernetes](https://ghoshasish99.medium.com/azure-devops-self-hosted-agents-on-kubernetes-51685fde9a14)

[https://blogs.zeiss.com/tech/ms-azure-devops-on-demand-agents/](https://blogs.zeiss.com/tech/ms-azure-devops-on-demand-agents/)

[https://blogs.zeiss.com/tech/ms-azure-devops-on-demand-agents-part-2/](https://blogs.zeiss.com/tech/ms-azure-devops-on-demand-agents-part-2/)