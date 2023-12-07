# TCP Services in Kubernetes

In Kubernetes, TCP services are used to expose applications running inside pods to external clients. A TCP service acts as a stable endpoint that can be accessed by other pods or external users. It provides load balancing functionality, distributing incoming traffic to the pods that are part of the service.

## Difference from NodePort

A NodePort is a type of service in Kubernetes that exposes an application outside the cluster by allocating a static port on each node. This means that the application can be accessed using the IP address of any node in the cluster, along with the allocated port number.

The main difference between a TCP service and a NodePort is the level of flexibility and scalability they provide. A TCP service offers advanced features like load balancing and automatic routing of traffic to the pods, ensuring high availability and efficient handling of increased traffic. On the other hand, a NodePort simply exposes the application on a specific port of each node, without any load balancing or routing capabilities.

For applications that require more advanced networking features and scalability, such as handling high traffic loads or distributing requests across multiple pods, a TCP service in Kubernetes would be a better choice. However, for simpler scenarios where direct access to a specific node's IP address and port is sufficient, a NodePort can be a suitable option.

## Currently Used TCP Ports

<table>
    <th>Service</th>
    <th>Port</th>
    <tr>
        <td>9094</td>
        <td>Kafka</td>
    </tr>
    <tr>
        <td>5672</td>
        <td>RabbitMQ</td>
    </tr>
    <tr>
        <td>15672</td>
        <td>RabbitMQ</td>
    </tr>
</table>