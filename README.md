# docker-monitoring

This is the first 2016 setup that works with the current latest versions of Google cAdvisor (v0.20.5), InfluxDB 0.9, Grafana working (latest) together. This was part of the work done to verify the PR https://github.com/google/cadvisor/pull/1040#issuecomment-176078952.

# Setup

Just use docker-compose to bring all up in a single machine for smoke test.

```
$ docker-compose up
$ docker-compose ps
             Name                            Command               State                                  Ports                                
----------------------------------------------------------------------------------------------------------------------------------------------
dockermonitoring_cadvisor_1       /usr/bin/cadvisor -logtost ...   Up       0.0.0.0:9090->8080/tcp                                             
dockermonitoring_grafana_1        /run.sh                          Up       0.0.0.0:3000->3000/tcp                                             
dockermonitoring_influxdbData_1   sh                               Exit 0                                                                      
dockermonitoring_influxdb_1       /run.sh                          Up       0.0.0.0:8083->8083/tcp, 0.0.0.0:8086->8086/tcp, 8090/tcp, 8099/tcp
```

The images pulled for this setup were as follows:

```
$ docker images | grep influx
tutum/influxdb                                            latest                        b022febf81e2        7 weeks ago         275.2 MB
$ docker images | grep cadvisor
google/cadvisor                                           v0.20.5                       cb4f76a7607a        47 hours ago        43.12 MB
$ docker images | grep grafana
grafana/grafana                                           latest                        539a07877d03        8 days ago          213.9 MB
```

## Verify InfluxDB

1. Just login to InfluxDB and verify everything is working. The credentials `root:root` can be used. 
2. Select the database `cadvisor` from the drop-down, which is automatically created during bootstrap.
3. Select the qury `SHOW MEASUREMENTS`, which will display all the `cadvisor` data.

![influx](http://s21.postimg.org/kmfqs0k5j/Screen_Shot_2016_02_05_at_12_37_20_PM.png)

## Setup Grafana Dashboards

There are a lot of good resources online to create dashboards. I followed https://www.brianchristner.io/how-to-setup-docker-monitoring/, and there are others, and I could verify that the latest version of Grafana could properly load the InfluxDB 0.9 measurements just fine.

1. Create a new datasource using InfluxDB 0.9, make sure to select it as the default.

![graf](http://s28.postimg.org/445358p3x/Screen_Shot_2016_02_05_at_12_37_49_PM.png)

2. Create the dashboard selecting which measurements you want. Notice that the measurements loaded by InfluxDB, shown in the previous section, are also loaded here.

![meas](http://s30.postimg.org/jnx7tnwj5/Screen_Shot_2016_02_05_at_12_36_52_PM.png)

3. Check the result of the graph.

![img](http://s14.postimg.org/3vnn2t9up/Screen_Shot_2016_02_05_at_12_35_21_PM.png)

# Debug

# InfluxDB Creates DB once

Influx will not re-create the DB twice.

```
influxdb_1 | => About to create the following database: cadvisor
influxdb_1 | => Database had been created before, skipping ...
```

The full logs is just like this:

```
influxdb_1 | 
influxdb_1 | [hinted-handoff]
influxdb_1 |   enabled = true
influxdb_1 |   dir = "/data/hh"
influxdb_1 |   max-size = 1073741824
influxdb_1 |   max-age = "168h"
influxdb_1 |   retry-rate-limit = 0
influxdb_1 |   retry-interval = "1s"
influxdb_1 | => Starting InfluxDB ...
influxdb_1 | => About to create the following database: cadvisor
influxdb_1 | => Database had been created before, skipping ...
influxdb_1 | exec influxd -config=${CONFIG_FILE}
influxdb_1 | 2016/02/05 20:15:18 InfluxDB starting, version 0.9.6.1, branch 0.9.6, commit 6d3a8603cfdaf1a141779ed88b093dcc5c528e5e, built 2015-12-10T23:40:23+0000
influxdb_1 | 2016/02/05 20:15:18 Go version go1.4.2, GOMAXPROCS set to 4
influxdb_1 | 
influxdb_1 |  8888888           .d888 888                   8888888b.  888888b.
influxdb_1 |    888            d88P"  888                   888  "Y88b 888  "88b
influxdb_1 |    888            888    888                   888    888 888  .88P
influxdb_1 |    888   88888b.  888888 888 888  888 888  888 888    888 8888888K.
influxdb_1 |    888   888 "88b 888    888 888  888  Y8bd8P' 888    888 888  "Y88b
influxdb_1 |    888   888  888 888    888 888  888   X88K   888    888 888    888
influxdb_1 |    888   888  888 888    888 Y88b 888 .d8""8b. 888  .d88P 888   d88P
influxdb_1 |  8888888 888  888 888    888  "Y88888 888  888 8888888P"  8888888P"
influxdb_1 | 
influxdb_1 | 2016/02/05 20:15:18 Using configuration at: /config/config.toml
```

# cAdvisor Writes

cAdvisor properly writes to InfluxDB 0.9.

```
influxdb_1     | [http] 2016/02/05 20:31:22 172.17.0.2 - root [05/Feb/2016:20:31:22 +0000] POST /write?consistency=&db=cadvisor&precision=&rp= HTTP/1.1 204 0 - cAdvisor/0.20.5 6b62082c-cc47-11e5-8010-000000000000 22.037754ms
```
