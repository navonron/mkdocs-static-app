In this page we'll learn how to setup Apache JMeter for performance tuning and testing of ActiveMQ.

## What is Apache JMeter

The Apache JMeter™ application is open source software, a 100% pure Java application designed to load test functional behavior and measure performance.

## Install Java JDK

JMeter require Java JDK, you can install using the winget package manager on Windows.

```
winget install -e --id Oracle.JDK.17
```

## JMeter Setup
### Installing JMeter

Go to the URL below and download the Zip file for `apache-jmeter-5.6.2.zip` under the binaries section.
https://jmeter.apache.org/download_jmeter.cgi

2. Unzip it into it's own directory
```
unzip apache-jmeter-5.6.2.zip
```

### Configure JMeter
#### Heap Size	

Modify current env variable `HEAP="-Xms1g -Xmx1g -XX:MaxMetaspaceSize=256m"` in the JMeter batch file
 
Check : https://jmeter.apache.org/usermanual/best-practices.html	

In the `bin` directory of the extracted there will be a file named `jmeter.bat`

Edit it and below the line `set JMETER_CMD_LINE_ARGS=%*` insert:

```
set HEAP=-Xms1g -Xmx2g -XX:MaxMetaspaceSize=256M	
```

The two lines together should look like:

```
set JMETER_CMD_LINE_ARGS=%*
set HEAP=-Xms1g -Xmx2g -XX:MaxMetaspaceSize=256M
```

### Run JMeter

run the jmeter batch file
```
jmeter.bat
```

## Test Plans

Inside the extracted JMeter folder create a folder named "testplans" for the test plan files (doesn't have to be in that folder but is more convenient)

```
mkdir testplans
```

Under the testplans directory add the below files: