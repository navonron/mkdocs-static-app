## Introduction

The Hawtio Reader tool as Flask app built in order to mitigate the issue with the ActiveMQ Hawtio Web UI ([https://hawt.io/](https://hawt.io/)).

The Production environment can be accessed here : [http://ibi-activemq-ui.iglb.intel.com/](http://ibi-activemq-ui.iglb.intel.com/)

## How does it works?

Hawtio Reader pulls the information about ActiveMQ from the ActiveMQ REST API, and displays it to the user in a friendly way (same as the Hawtio Web UI).

The index page of the tools shows the list of all ActiveMQ hosts that are defined in the JSON configuration file of Hawtio Reader.

Each ActiveMQ host in the index page is a clickable link that leads to what replaces the Hawtio panel (Statistics about the ActiveMQ host, Table of all the queues and their properties).

## Additional Properties
In addition to that, the Hawtio Reader also adds statistics about important things, such as:

- Inactive Consumer Count
- Inactive Producer Count
- Active Consumer Count
- Active Producer Count
- Broker Throughput (The number of messages per second for the entire broker)
- Queue throughput (Present in the Queues table with the Throughput of each queue)

## Capabilities
It also packs capabilities:

1. Read Data from the Hawtio REST API on ActiveMQ broker (using JMX of Jolokia).
2. Read Data from the ActiveMQ REST API on ActiveMQ broker (using JMX of Jolokia).
3. Auto Refresh (default is without, can be set from the side panel).


## Code
The Hawtio Reader code is stored in this Azure DevOps git repo: [https://dev.azure.com/PDA-PDSS/iBI/_git/amq-hawtio-reader](https://dev.azure.com/PDA-PDSS/iBI/_git/amq-hawtio-reader)


## Pipeline
The amq-hawtio-reader repo also packs an Azure Pipeline file that automatically deploys the recent version of hawtio reader to Kubernetes in GER upon any code changes.

## Closing words
Usually the Hawtio Pod can be found in Kubernetes under the daas project, and then under "workloads" [https://ger.caas.intel.com/p/c-xx6ck:p-grj9r/workloads](https://ger.caas.intel.com/p/c-xx6ck:p-grj9r/workloads).