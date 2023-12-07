A service principal name (SPN) is a unique identifier of a service instance.

SPNs are used by Kerberos authentication to associate a service instance with a service logon account.

This allows a client application to request that the service authenticate an account even if the client does not have the account name.

If you install multiple instances of a service on computers throughout a forest, each instance must have its own SPN. A given service instance can have multiple SPNs if there are multiple names that clients might use for authentication. For example, an SPN always includes the name of the host computer on which the service instance is running, so a service instance might register an SPN for each name or alias of its host.


### SPN format and composing a unique SPN:

An SPN must be unique in the forest in which it is registered. If it is not unique, authentication will fail. The SPN syntax has four elements: 2 required elements and 2 additional optional elements that you can use, if necessary, to produce a unique name as listed in the following table.

```
`<service` `class``>/<host>:<port>/<service name>`
```


|"service class"|A string that identifies the general class of service; for example, "SqlServer".|
|---|---|
|"host"|The name of the computer on which the service is running.|
|"port"|An optional port number to differentiate between multiple instances of the same service class on a single host computer.|
|"service name"|An optional name used in the SPNs of a replicable service to identify the data or services provided by the service or the domain served by the service.|

For more information please refer to the following link [Name Formats for Unique SPNs](https://docs.microsoft.com/en-us/windows/win32/ad/name-formats-for-unique-spns).

Before the Kerberos authentication service can use an SPN to authenticate a service, the SPN must be registered on the account

object that the service instance uses to log on.

A given SPN can be registered on only one account. For Win services, a service installer specifies the logon account when an instance of the service is installed.

The installer then composes the SPNs and writes them as a property of the account object in Active Directory Domain Services.

If the logon account of a service instance changes, the SPNs must be re-registered under the new account.

When a client wants to connect to a service, it locates an instance of the service, composes an SPN for that instance, connects to the service, and presents the SPN for the service to authenticate. For more information, see How Clients Compose a Service's SPN.
