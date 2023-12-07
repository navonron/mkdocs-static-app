# OpenLens

## **Background**

[OpenLens](https://github.com/lensapp/lens) is an open source desktop application developed to allow users and view and administer Kubernetes clusters.

## **Installation**

To install OpenLens you can use the winget tool on Windows
```
winget install openlens
```

## **Extensions**

OpenLens can be extended with many useful features, the extensions I found were best are the below

### **OpenLens Node/Pod Menu Extension**
This OpenLens extension adds back the node and pod menu functionality that was removed from OpenLens itself in 6.3.0.

This extensions allows you to:

1. Get node shell
2. Drain a node
3. Cordon a node

[Repo Link](https://github.com/alebcay/openlens-node-pod-menu)

```
@alebcay/openlens-node-pod-menu
```

### **Multi Pod Logs Lens Extension**

A Lens (or OpenLens) extension that enables you to see logs from multiple pods (and multiple containers within the pod) on Kubernetes.

[Repo Link](https://github.com/andrea-falco/lens-multi-pod-logs)

```
@andrea-falco/lens-multi-pod-logs
```


### **Kubescape Lens Extension**

A Lens extension for viewing [Kubescape](https://github.com/kubescape/kubescape) security information

*Kubescape is an open-source Kubernetes security platform for your IDE, CI/CD pipelines, and clusters

Kubescape is an open-source Kubernetes security platform. It includes risk analysis, security compliance, and misconfiguration scanning. Targeted at the DevSecOps practitioner or platform engineer, it offers an easy-to-use CLI interface, flexible output formats, and automated scanning capabilities. It saves Kubernetes users and admins precious time, effort, and resources.

Kubescape scans clusters, YAML files, and Helm charts. It detects misconfigurations according to multiple frameworks (including [NSA-CISA](https://www.armosec.io/blog/kubernetes-hardening-guidance-summary-by-armo/?utm_source=github&utm_medium=repository), [MITRE ATT&CK®](https://www.microsoft.com/security/blog/2021/03/23/secure-containerized-environments-with-updated-threat-matrix-for-kubernetes/) and the [CIS Benchmark](https://www.armosec.io/blog/cis-kubernetes-benchmark-framework-scanning-tools-comparison/?utm_source=github&utm_medium=repository)).

Kubescape was created by [ARMO](https://www.armosec.io/?utm_source=github&utm_medium=repository) and is a [Cloud Native Computing Foundation (CNCF) sandbox project](https://www.cncf.io/sandbox-projects/).*

[Repo Link](https://github.com/kubescape/lens-extension)

```
@kubescape/lens-extension
```