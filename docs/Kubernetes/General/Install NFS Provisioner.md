# Install NFS Client Provisioner

## Clone the public git repo

We first clone the nfs-subdir-external-provisioner with branch version 4.0.15 (as recommended by CaaS).
```shell
git clone https://github.com/kubernetes-sigs/nfs-subdir-external-provisioner --branch nfs-subdir-external-provisioner-4.0.15
cd nfs-subdir-external-provisioner/charts
```

## Adding securityContext
One of the parameters we need to add to the values.yaml is the securityContext for the NFS storage.
That way volume will be create with the permissions of our user instead of root.

```yaml
securityContext:
  runAsUser: 33490 # DesignML
  runAsGroup: 18705 # pdaunix
```

## Adding NFS Server and path

### GER
```yaml
nfs:
  server: "iilc11n01a.iil.intel.com"
  path: "/site_disks_designml_tools/caas/shared_storage"
```

### AMR
```yaml
nfs:
  server: "pdxcfsv09a.pdx.intel.com"
  path: "/nfs/pdx/disks/pda_ml"
```

## Other 
```yaml
storageClass:
  defaultClass: true
```

```shell
​
# It is recommended to install it in a dedicated / separate namespace, like nfs-client but this is not a hard requirement.
​
helm install --create-namespace --version 4.0.15 --namespace [NAMESPACE] --set nfs.server="[NFS SERVER]" --set nfs.path="[NFS PATH]" --set storageClass.defaultClass=[true or false] [HELM CHART NAME] ./nfs-subdir-external-provisioner/
​
​# Example
# You can also point to the private harbor registry if you know that the image is available there
# --set image.repository=(ger|amr)-registry.caas.intel.com/<path…>/nfs-subdir-external-provisioner
​
helm install --create-namespace --version 4.0.15 --namespace nfs-client nfs-client ./nfs-subdir-external-provisioner/ --kubeconfig "C:\Users\<user-name>\AppData\Roaming\OpenLens\kubeconfigs\67048fdb-052b-4d91-a0c6-bcddc61c8838" --wait --dry-run
helm upgrade --install --create-namespace --version 4.0.15 --namespace nfs-client nfs-client ./nfs-subdir-external-provisioner/ --kubeconfig "C:\Users\<user-name>\AppData\Roaming\OpenLens\kubeconfigs\67048fdb-052b-4d91-a0c6-bcddc61c8838" --wait
```