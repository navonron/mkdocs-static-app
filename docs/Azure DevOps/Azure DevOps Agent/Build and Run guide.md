## About

Azure DevOps Runners (will also be referred to as “ADO Runner”) is a Dockerized Azure DevOps Agent on Azure DevOps, that is platform agnostic, offering the same capabilities of Azure DevOps Cloud Agents (Clean agent environment) and with the ability to access both Cloud Subscriptions resources AND On Premise resources at the same time

It is based on the Ubuntu 18.04 Docker image, but can be migrated to other types (including Windows) with additional development effort.

ADO runners can run on any environment as long as it has Docker installed (e.g. Laptop with Docker, Kubernetes, etc.) .

## Build ADO Runners

1. Clone the ADO Runners repository from Azure DevOps
2. Use the command below (PowerShell) to build the image locally.

💡 Take note that the `http_proxy` and `https_proxy` are not needed if running without VPN.


```powershell
# Define proxy for the docker build command
$HTTP_PROXY = "http://proxy-chain.intel.com:911"
$HTTPS_PROXY = "http://proxy-chain.intel.com:912"
$IMAGE_NAME = "azure-devops-runner"
$IMAGE_TAG = "latest"

# Build the docker image
docker build --build-arg http_proxy=$HTTP_PROXY --build-arg https_proxy=$HTTPS_PROXY -t $IMAGE_NAME:$IMAGE_TAG -f .\Dockerfile .
```

  
After the Docker image is built you can push it to your preferred Docker registry.

## Usage

The Azure DevOps Runner can be used in multiple ways.
## Run ADO Runner as Docker image

To run the ADO Runners locally, you need to provide the below parameters, and then run the docker image:


```powershell
$AZURE_AGENT_POOL = "" 
$AZURE_AGENT_URL = ""
$AZURE_AGENT_TOKEN = ""
$AZURE_AGENT_NAME = "azure-agent-docker"
$IMAGE_NAME = "azure-devops-runner"
$IMAGE_TAG = "latest"

docker run -e AZP_POOL=$AZURE_AGENT_POOL -e AZP_URL=$AZURE_AGENT_URL -e AZP_TOKEN=$AZURE_AGENT_TOKEN -e AZP_AGENT_NAME=$AZURE_AGENT_NAME $IMAGE_NAME:$IMAGE_TAG -v /var/run/docker.sock:/var/run/docker.sock
```

## Run ADO Runner in Kubernetes
 
To run ADO Runners in Kubernetes build a yaml file for a “deployment” or a “statefulset”

💡 In order for Docker operations to work you need to mount the docker.sock from the host to the ADO Runner container (using volume as shown about in the run section).

Run the command:

```powershell
# Yaml can be of a deployment or a statefulset.
kubectl apply -f deployment.yaml
```