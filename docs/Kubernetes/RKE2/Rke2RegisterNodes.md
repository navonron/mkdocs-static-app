# Register RKE2 Nodes in Rancher

When registering a new Rancher node you must configure SPN for that node so that Kerberos applications will be able to run on it.

[Service Principal Names (SPN)](../../OnPrem/Service%20Principal%20Name%20(SPN)/Service%20Principal%20Names%20(SPN).md)

## **Cleaning up RKE1 docker server**
Previously installed servers with RKE1 which use Docker at their runtime needs to be cleaned up properly before migrating the the new RKE2 cluster.
Follow the below steps to complete the cleanup:

### **Show Docker system information**
This command will look up how much space is used (-v is used for verbose)
You'll get information about:

* Images space usage,
* Containers space usage,
* Local Volumes space usage, and
* Build cache usage.

``` bash
docker system df -v
```

### **Cleanup**
IF you want to delete all the Docker information from the system including the running components
run this command to stop all the containers
``` bash
docker stop $(docker ps -a -q)
```

Run the below commands to clean up as much as possible excluding components that are in use:
``` bash
docker system prune -af
docker volume prune -f
```

## **Registration command - Worker Node**
``` shell
http_proxy="http://proxy-dmz.intel.com:912" https_proxy="http://proxy-dmz.intel.com:912" HTTP_PROXY="http://proxy-dmz.intel.com:912" HTTPS_PROXY="http://proxy-dmz.intel.com:912" NO_PROXY="localhost,127.0.0.0/8,127.0.0.1/8,10.0.0.0/8,.intel.com" no_proxy="localhost,127.0.0.0/8,127.0.0.1/8,10.0.0.0/8,.intel.com" curl -fL https://ger-pre.caas.intel.com/system-agent-install.sh | http_proxy="http://proxy-dmz.intel.com:912" https_proxy="http://proxy-dmz.intel.com:912" HTTP_PROXY="http://proxy-dmz.intel.com:912" HTTPS_PROXY="http://proxy-dmz.intel.com:912" NO_PROXY="localhost,127.0.0.0/8,127.0.0.1/8,10.0.0.0/8,.intel.com" no_proxy="localhost,127.0.0.0/8,127.0.0.1/8,10.0.0.0/8,.intel.com" sh -s - --server https://ger-pre.caas.intel.com --label 'cattle.io/os=linux' --token 5ffvvrn66lnb8q5s6vwqtmhbkfnd5dtgt7sgqgnvpq8v57cg5wpq84 --ca-checksum c398aa1af0fe535743ffdddc58aa36080fc106251329d3367c55d59d9338bc61 --worker
```

## **Registration command - Etcd & ControlPlane Nodes**
``` shell
http_proxy="http://proxy-dmz.intel.com:912" https_proxy="http://proxy-dmz.intel.com:912" HTTP_PROXY="http://proxy-dmz.intel.com:912" HTTPS_PROXY="http://proxy-dmz.intel.com:912" NO_PROXY="localhost,127.0.0.0/8,127.0.0.1/8,10.0.0.0/8,.intel.com" no_proxy="localhost,127.0.0.0/8,127.0.0.1/8,10.0.0.0/8,.intel.com" curl -fL https://ger-pre.caas.intel.com/system-agent-install.sh | http_proxy="http://proxy-dmz.intel.com:912" https_proxy="http://proxy-dmz.intel.com:912" HTTP_PROXY="http://proxy-dmz.intel.com:912" HTTPS_PROXY="http://proxy-dmz.intel.com:912" NO_PROXY="localhost,127.0.0.0/8,127.0.0.1/8,10.0.0.0/8,.intel.com" no_proxy="localhost,127.0.0.0/8,127.0.0.1/8,10.0.0.0/8,.intel.com" sh -s - --server https://ger-pre.caas.intel.com --label 'cattle.io/os=linux' --token 5ffvvrn66lnb8q5s6vwqtmhbkfnd5dtgt7sgqgnvpq8v57cg5wpq84 --ca-checksum c398aa1af0fe535743ffdddc58aa36080fc106251329d3367c55d59d9338bc61 --etcd --controlplane
```

## **How to cleanup a node from RKE2 installation**
``` bash
cd /usr/local/bin/
./rke2-killall.sh
./rke2-uninstall.sh
systemctl stop rancher-system-agent.service
systemctl disable rancher-system-agent.service
systemctl reset-failed rancher-system-agent.service
rm -rf /etc/systemd/system/rancher-system-agent.*
systemctl daemon-reload
systemctl reset-failed
rm -f /usr/local/bin/rancher-system-agent
rm -rf /etc/rancher
rm -rf /var/lib/rancher/
rm -rf /usr/local/bin/rke2*
```

## **Services**
``` bash
systemctl status rancher-system-agent.service -l
systemctl restart rancher-system-agent.service
journalctl -u rancher-system-agent -xe -f
```

## **Containerd commands**
[RKE2 Commands](./Rke2Commands.md)