## Introduction

ActiveMQ is an open source, multi-protocol, Java-based message broker.

It supports industry standard protocols so users get the benefits of client choices across a broad range of languages and platforms. Connect from clients written in JavaScript, C, C++, Python, .Net, and more.

Integrate your multi-platform applications using the ubiquitous AMQP protocol. Exchange messages between your web applications using STOMP over websockets. Manage your IoT devices using MQTT.

ActiveMQ sends messages between client applications producers, which create messages and submit them for delivery, and consumers, which receive and process messages.

Project Page:  [https://activemq.apache.org/](https://activemq.apache.org/) 

## How does it work?

The ActiveMQ broker routes each message through a messaging endpoint called a destination (in ActiveMQ Classic).

ActiveMQ Classic is capable of point-to-point messaging in which the broker routes each message to one of the available consumers in a round robin pattern and publish/subscribe (or pub/sub) messaging in which the broker delivers each message to every consumer that is subscribed to the topic.

![](ActiveMQ%201.png)
