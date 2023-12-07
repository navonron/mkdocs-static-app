ActiveMQ in our current production environments is configured to be run as a service.
This service auto starts when the ActiveMQ boots and CMCC team has monitoring that will restart it if it ever fails.

In theory, there shouldn't be any reason for manual intervention anymore. but just in case here are two ways to start the ActiveMQ broker.

## ActiveMQ Service

The commands below shows how to start the current ActiveMQ broker on our physical clusters.

1. SSH into the ActiveMQ host (Example: `icsl6673.iil.intel.com`)
```shell
ssh <ACTIVEMQ_HOST>.<SITE>.intel.com
```

2. Switch into the root account (or another account that has permissions to run the service)
```shell
sudo rootsh
```

3. Run the ActiveMQ Service
```shell
systemctl start activemq.service
```

## Running Process Manually

In case the ActiveMQ service does not exist on the host, here are the commands to start it manually.

> ⛔ **These commands needs to run as root** -as they increase File Descriptor Limit & Max PIDs:

```bash
#!/bin/bash
ulimit -n 65536
echo 4194303 > /proc/sys/kernel/pid_max
/bin/su ibiuser -c "/infrastructure/apache-activemq-5.16.2/bin/activemq start"
```

> ⛔ User tasks maximum should also be increased for ActiveMQ to run properly. 