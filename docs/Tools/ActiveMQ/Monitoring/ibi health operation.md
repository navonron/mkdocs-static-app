The ActiveMQ health operation was built specifically for ActiveMQ in order to make sure it's up to a certain number of standards.

General command show the usage:

```
ibi.exe health get usage --domain=qos --operation=activemq
```

To check the health of a site, `--params="site=SITE_NAME"` to the command: 

```shell
# Checking the OR site
ibi.exe health check --domain=qos --operation=activemq --params="site=OR"

# Checking the IIL site
ibi.exe health check --domain=qos --operation=activemq --params="site=IIL"

# Checking the FM site
ibi.exe health check --domain=qos --operation=activemq --params="site=FM"

# Checking the SC site
ibi.exe health check --domain=qos --operation=activemq --params="site=SC"
```