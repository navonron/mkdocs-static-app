This page will detail the information about ActiveMQ's memory management.
## Memory Usage

This is the % of assigned memory of the broker that has been used up to keep track of destinations, cache messages etc. This value needs to be lesser than -Xmx  (Max JVM heap size)

By default it's set to be 70%..

On our Linux machines with 64GB of RAM the heap size for ActiveMQ is set to 35GB.

This parameter is defined in the file: `/infrastructure/apache-activemq-5.16.2/bin/env`
![](Memory%20Management%201.png)

Since the broker is configured to utilize 70% of that capacity is can use approximately 24.5GB.

This can always be expanded but currently is deemed sufficient, taking into account the observed RAM usages.

```xml
<memoryUsage>
	<memoryUsage percentOfJvmHeap="70" />
</memoryUsage>
```

## Storage Usage

Storage Usage is the amount of disk space that ActiveMQ is allowed to use to store persistent messages.

```xml
<storeUsage>
	<storeUsage limit="100 gb"/>
</storeUsage>
```

## TempUsage

This is the % of assigned disk storage that has been used up to spool non-persistent messages.
Non persistent messages are those that don’t survive broker restart

```xml
<tempUsage>
	<tempUsage limit="25 gb"/>
</tempUsage>
```

The SystemUsage object has a MemoryUsage, StoreUsage, and TempUsage associated with it.