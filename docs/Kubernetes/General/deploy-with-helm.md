# Deploy to K8s with Helm

## **Adding PDA Repo To List**
Add the PDA repository to the Helm repo list using the command below

```shell
helm repo add harbor https://ger-is-registry.caas.intel.com/chartrepo/pda --insecure-skip-tls-verify --username <USERNAME> --password <PASSWORD> --debug
```

## **Checking PDA Repo Is Added**
You can then use `helm repo list` to check if the repo is added
```
[C:\Work] $ helm repo list
NAME    URL
harbor  https://ger-is-registry.caas.intel.com/chartrepo/pda
```

## **List All Installed Apps**
```shell
helm list --all-namespaces
```

## **Show Info About a chart**
```shell
helm show all harbor/ibi-service
```

## **Installing a Chart to a Cluster**
To install a helm chart using the helm cli you need to specify some parameters:

1. Chart Name in Kubernetes
2. Chart Path
3. Namespace where the App will be installed
4. Value file path - Where the values file resides.
5. Kubeconfig - The path to the kubeconfig file (to know which cluster to install it on)

```shell
helm upgrade --install <CHART_NAME> <CHART_PATH> --insecure-skip-tls-verify -n <NAMESPACE> --atomic --create-namespace -f <VALUE_YAML_PATH> --kubeconfig <KUBECONFIG_PATH> --debug
```

Example of installing the **Hawtio Reader** App in the `ger-pre-pda-cluster` in the CaaS Pre Prod environment.
```shell
helm upgrade --install hawtioreader harbor/ibi-service --insecure-skip-tls-verify -n infrastructure --atomic --create-namespace -f ./values.yaml --kubeconfig C:/Users/$env:username/.kube/ger-pre-pda-cluster.yaml --debug
```