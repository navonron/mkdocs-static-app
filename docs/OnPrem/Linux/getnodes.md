# getnodes help


Get a list of all iBI Servers (Filter nodes by `speciality~=iBI-Server || speciality~=DesignML-Server`).

```bash
getnodes --fi hostname,location,osver,owner,speciality::40 "speciality~=iBI-Server || speciality~=DesignML-Server" -cs --sort hostname --format table
```

Show Hostname, RAM, Core Count, Location, OS Version, Owner and Specialties of the current server
```shell
getnodes --fi hostname,memory,cores,location,osver,owner,speciality -cs --sort hostname --format csv
```

```bash
getnodes --default-fields
```

```bash
getnodes --set-format table
```

```bash
getnodes --set-field hostname,location,osver,owner,speciality,mode
```

Link to ComputeBI
[Link](https://computebi.swiss.intel.com/results/all/(speciality%20CONTAINS%20iBI-Server)%20OR%20(speciality%20CONTAINS%20DesignML-Server))