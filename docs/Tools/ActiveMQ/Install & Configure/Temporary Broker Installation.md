This is a guide on how to setup a temporary broker installation for cases where a broker needs to be setup as soon as possible.

This page was originally created because of the shutdown in Oregon on August 2023 where the entire datacenters (JF4 & JFS1) were down for a week and a half and we had to setup ActiveMQ in another site an FAST.

## Host Requirements

Before you begin, ensure that your host machine meets the following requirements:

- 1GB Ethernet
- 4 CPU cores
- 24GB RAM
- 100GB Disk space (recommended, to avoid potential bottlenecks)

## Installing Docker on the Linux Host

To install Docker on your Linux host, follow these steps:

1. Go to Azure DevOps and copy the `install_docker.sh` script for SLES 12 SP5.

2. Switch to the root user using one of the following commands, depending on your system:

   For VRA Linux machine:
   ```shell
   sudo su
   ```

   For HPC Linux machine:
   ```shell
   sudo rootsh
   ```

3. Use a text editor like `jpico` to paste the contents of the `install_docker.sh` script and make it executable:

   ```shell
   jpico ./install_docker.sh
   ```

4. Copy the contents, save the file, and make it executable:

   ```shell
   chmod +x ./install_docker.sh
   ```

5. Run the script to install Docker:

   ```shell
   ./install_docker.sh
   ```

6. Add the Docker group if it doesn't exist:

   ```shell
   groupadd docker
   ```

7. Add your user to the Docker group (Should still be root from the commands above):

   ```shell
   usermod -aG docker $USER
   ```

8. Create a user for ActiveMQ actions and add it to the Docker group:

   ```shell
   useradd -m activemq
   usermod -aG docker activemq
   ```

9. Switch to the ActiveMQ user:

   ```shell
   su - activemq
   ```

10. Create a directory for persistent data under the activemq user home directory (or use NFS):

   ```shell
   mkdir -p /infrastructure/activemq
   ```

## Initializing ActiveMQ Configuration

ActiveMQ expects that some configuration files already exist. You can initialize your directories using an intermediate container. Run the following commands inside the intermediate container to copy the default configuration to your host directory:

Command to run the intermediate container:
    ```
    docker run --user root --rm -ti \
    -v /infrastructure/activemq/conf:/mnt/conf \
    -v /infrastructure/activemq/data:/mnt/data \
    rmohr/activemq:5.15.4-alpine /bin/sh
    ```

Commands to run inside the intermediate container:
    ```
    cp -a /opt/activemq/conf/* /mnt/conf/
    cp -a /opt/activemq/data/* /mnt/data/
    exit
    ```


You'll also need to replace the /mnt/conf/activemq.xml with the text below:
```xml
<beans
  xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
  http://activemq.apache.org/schema/core http://activemq.apache.org/schema/core/activemq-core.xsd">

    <bean class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
        <property name="locations">
            <value>file:${activemq.conf}/credentials.properties</value>
        </property>
    </bean>

    <bean id="logQuery" class="io.fabric8.insight.log.log4j.Log4jLogQuery"
          lazy-init="false" scope="singleton"
          init-method="start" destroy-method="stop">
    </bean>

    <broker xmlns="http://activemq.apache.org/schema/core" brokerName="localhost" useJmx="true" dataDirectory="${activemq.data}">

        <destinationPolicy>
            <policyMap>
              <policyEntries>
                <policyEntry topic=">" >
                  <pendingMessageLimitStrategy>
                    <constantPendingMessageLimitStrategy limit="1000"/>
                  </pendingMessageLimitStrategy>
                </policyEntry>
              </policyEntries>
            </policyMap>
        </destinationPolicy>

        <plugins>
            <discardingDLQBrokerPlugin dropAll="true" dropTemporaryTopics="true" dropTemporaryQueues="true"/>
        </plugins>
        <managementContext>
            <managementContext createConnector="true" connectorPort="1099"/>
        </managementContext>

        <persistenceAdapter>
            <kahaDB directory="${activemq.data}/kahadb"/>
        </persistenceAdapter>
          <systemUsage>
            <systemUsage>
                <memoryUsage>
                    <memoryUsage percentOfJvmHeap="70" />
                </memoryUsage>
                <storeUsage>
                    <storeUsage limit="25 gb"/>
                </storeUsage>
                <tempUsage>
                    <tempUsage limit="25 gb"/>
                </tempUsage>
            </systemUsage>
        </systemUsage>

        <transportConnectors>
            <transportConnector name="openwire" uri="tcp://0.0.0.0:61616?wireFormat.maxFrameSize=524288000"/>
        </transportConnectors>

        <!-- destroy the spring context on shutdown to stop jetty -->
        <shutdownHooks>
            <bean xmlns="http://www.springframework.org/schema/beans" class="org.apache.activemq.hooks.SpringContextHook" />
        </shutdownHooks>

    </broker>
    <import resource="jetty.xml"/>
</beans>
```

For a single independent container the only things that change are:

The plugins section is added that drops temporary queues and topics and discard DLQ:
```xml
<plugins>
    <discardingDLQBrokerPlugin dropAll="true" dropTemporaryTopics="true" dropTemporaryQueues="true"/>
</plugins>
```

`useJmx="true"` is added to the broker tag to enable JMX
```xml
<broker xmlns="http://activemq.apache.org/schema/core" brokerName="localhost" useJmx="true" dataDirectory="${activemq.data}">
```

Now that Docker is set up and the configuration is ready, you can pull and run the ActiveMQ Docker container:

```shell
docker pull rmohr/activemq:latest
# Change ownership to the amq directory (since the intermediate container will create the data & conf directories with different permissions which will cause issues)
#chown -R activemq:users /home/activemq/amq/*
chmod -R a+rw *

# Command pre
#echo 65536 > /sys/fs/cgroup/pids/user.slice/user-11925248.slice/pids.max
echo 4194303 > /proc/sys/kernel/pid_max
ulimit -n 65536

# Run the docker with one volume for the data (persistence), second volume for configuration files
docker run -d --name activemq_broker --pids-limit 4194303 --ulimit nofile=65536:65536 --rm -p 61616:61616 --env ACTIVEMQ_OPTS="-Xms2G -Xmx60G" -p 8161:8161 -v /infrastructure/activemq/conf:/opt/activemq/conf -v /infrastructure/activemq/data:/opt/activemq/data rmohr/activemq:latest
```

## Docker Run Command Breakdown

This command starts a detached Docker container named activemq_broker, limits the number of processes, sets ulimit for file descriptors, removes the container when it stops, runs it as the activemq user, maps host ports to container ports, and mounts host directories for configuration and data.

The following command is used to run a Docker container for ActiveMQ with various options and configurations:

```bash
docker run \
-d \                          # Run the container in detached mode (in the background).
--name activemq_broker \      # Assign a name to the container (activemq_broker).
--pids-limit 4194303 \        # Set a limit on the number of processes inside the container.
--ulimit nofile=65536:65536 \ # Set ulimit for the maximum number of file descriptors.
--rm \                        # Automatically remove the container when it stops.
--user activemq \             # Set the user to run the container as (activemq).
-p 61616:61616 \              # Map port 61616 from the host to port 61616 in the container.
-p 8161:8161 \                # Map port 8161 from the host to port 8161 in the container.
-v /infrastructure/activemq/conf:/opt/activemq/conf \ # Mount the host directory /infrastructure/activemq/conf to /opt/activemq/conf in the container.
-v /infrastructure/activemq/data:/opt/activemq/data \ # Mount the host directory /infrastructure/activemq/data to /opt/activemq/data in the container.
rmohr/activemq:latest                # The name of the Docker image to use (rmohr/activemq).
```


Check the logs of the ActiveMQ broker for errors using the command: 

```shell
docker logs activemq_broker -t -f
```

To stop the ActiveMQ broker:

```
docker stop activemq_broker
```

To restart the ActiveMQ broker:

```
docker restart activemq_broker
```
## Port Mapping

- 61616: JMS
- 8161: UI
- 5672: AMQP (since `rmohr/activemq:5.12.1`)
- 61613: STOMP (since `rmohr/activemq:5.12.1`)
- 1883: MQTT (since `rmohr/activemq:5.12.1`)
- 61614: WS (since `rmohr/activemq:5.12.1`)


```
java -XX:+PrintFlagsFinal -version | grep HeapSize
```

## Installing monitoring using metricbeat

After this is done please refer to the [Temporary Monitoring Installation](Temporary%20Monitoring%20Installation.md) guide.
