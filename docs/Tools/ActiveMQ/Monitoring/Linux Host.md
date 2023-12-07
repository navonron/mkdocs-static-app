Monitoring ActiveMQ cannot be done directly from the Linux host, but we can use the Linux command line in order to view server related metrics.

## Monitoring CPU, RAM, and Disk Usage on Linux

Monitoring the performance of your Linux system is crucial for maintaining its health and efficiency.

## Checking CPU Usage

To check CPU usage, you can use the `top` or `htop` command. 

Open your terminal and run:

```shell
top
```

or

```shell
htop
````

These commands will display a real-time view of processes and their CPU consumption. Press `q` to exit.

## Checking RAM Usage

For monitoring RAM usage, use the `free` command:

```shell
free -h
```
This will show you information about total, used, and available memory in human-readable format.

## Checking Disk Usage

To check disk usage, the `df` command is your go-to option:

```shell
df -h
```

This command will display information about disk space usage on all mounted filesystems.
The important on to us is usually the `/infrastructure` partition.