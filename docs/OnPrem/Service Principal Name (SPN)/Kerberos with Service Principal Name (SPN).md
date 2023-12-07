Kerberos is a protocol for authenticating service requests between trusted hosts across an untrusted network.

Kerberos support is built in to all major computer operating systems, including Microsoft Windows and Linux.

Kerberos protocol as the default authentication method in Windows, and it is an integral part of the Windows [ActiveDirectory](ActiveDirectory.md) (AD) service. 
Kerberos (Cerberus) was a three-headed dog who guarded the gates of Hades. The three heads of the Kerberos protocol represent the following:

1. The client or principal;
2. The network resource, which is the application server that provides access to the network resource; and
3. a key distribution center (KDC), which acts as Kerberos' trusted third-party authentication service.

Kerberos (Cerberus) was a three-headed dog who guarded the gates of Hades. The three heads of the Kerberos protocol represent the following:

1. The client or principal;
2. The network resource, which is the application server that provides access to the network resource; and
3. a key distribution center (KDC), which acts as Kerberos' trusted third-party authentication service.

Users, systems and services using Kerberos need only trust the KDC. It runs as a single process and provides two services: an authentication service and a ticket granting service (TGS).

![](Kerberos%20with%20Service%20Principal%20Name%20(SPN)%201.png)

To use Kerberos authentication with a Server requires both the following conditions to be true:

- The client and server computers must be part of the same Windows domain, or in trusted domains.
- A Service Principal Name (SPN) must be registered with Active Directory, which assumes the role of the Key Distribution Center in a Windows domain. The SPN, after it's registered, maps to the Windows account that started the server instance service. If the SPN registration hasn't been performed or fails, the Windows security layer can't determine the account associated with the SPN, and Kerberos authentication isn't used.

If the server can't automatically register the SPN, the SPN must be registered manually. See Manual SPN Registration.

[Service Principal Names (SPN)](Service%20Principal%20Names%20(SPN).md)
## Issue with LINUX servers

Service Principal Names (SPN) is a unique identifier for each service. We must have an SPN for each instance. In the case of multiple instances, we must register all the SPN. It is a mandatory step for a server connections to use Kerberos authentication.

For all services, the format of SPN is HTTP/<server address>@<server domain> (e.g. HTTP/icsl6660.iil.intel.com@GER.CORP.INTEL.COM). In order to check whether an SPN is registered, the following command can be used:

![](Kerberos%20with%20Service%20Principal%20Name%20(SPN)%202.png)

In a server(Win/LINUX) with SPN configured there is no problem and with the authentication process, we can check that by running the following command to authenticate a certain host with the authentication server:

![](Kerberos%20with%20Service%20Principal%20Name%20(SPN)%203.png)

However, opposing to Wins server Linux servers required to manually configured SPN for each host individually, else it will fail to authenticate.

Currently, most of the hosts that iBI use are configured, but this issue will occur when new Linux hosts will be spawned, this mean when setting them up will require a step of configuring a SPN.

To test it out, please run the following command from any Win/Linux server you wish to check if the authentication works:

On Linux:
```shell
curl -k --negotiate -u : https://ibi-daas-api-dev.intel.com/login
```

On Windows:
```shell
curl.exe -k --negotiate -u : https://ibi-daas-api-dev.intel.com/login
```

The iBI services authentication flow process can seen in the following chart:

![](Kerberos%20with%20Service%20Principal%20Name%20(SPN)%204.png)


## Creating a new SPN (Create SPN)

Please refer to the instructions here in order to submit a new request: [https://wiki.ith.intel.com/display/ActiveDirectory/Kerberos](https://wiki.ith.intel.com/display/ActiveDirectory/Kerberos)

It is important to register the SPNs for the specific faceless account **AMR\sys_aibi**, e.g.:

![](Kerberos%20with%20Service%20Principal%20Name%20(SPN)%205.png)

Once the request is completed, the registered machines can participate in Kerberos authentication process (e.g. be added to the CaaS Kubernetes cluster).