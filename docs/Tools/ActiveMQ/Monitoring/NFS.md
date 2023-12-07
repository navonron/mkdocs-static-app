Our ActiveMQ installations are using NFS as storage for the ActiveMQ data files.

To monitor the NFS Storage we use Intel "START".

"START" is an Intel system for managing storage.

In this case, we can use it to check the status of the ActiveMQ NFS mountpoint in the following link: [https://startweb.intel.com/dashboards/areas/iil/iil.345327](https://startweb.intel.com/dashboards/areas/iil/iil.345327) (IIL NFS)

In this link you can find the usage & size of the ActiveMQ share + the number of files in it as well.

![](NFS%201.png)
![](NFS%202.png)

There is also another link to Grafana of HPC with more statistics (not needed for ActiveMQ but just in case) : [Grafana - Volume (intel.com)](http://isec_storage_grafana.intel.com:9000/d/7ibs98Zmk/volume?orgId=1&var-cell=iil&var-cluster=iilc10.iil.intel.com&var-cluster_node=iilc10n05a)
