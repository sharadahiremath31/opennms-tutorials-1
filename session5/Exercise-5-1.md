
[Main Menu](../README.md) | [Session 5](../session5/README.md) | [Exercise-5-1](../session5/Exercise-5-1.md)

# Exercise 5-1 Accessing the sample network

## Introduction

We have an SNMP enabled device which provides the system load as a metric to indicate the system CPU and I/O utilization.
The metric is provided under the following SNMP object ids:

```
snmpwalk -On -v2c -c public localhost:1610 .1.3.6.1.4.1.2021.10.1.5
.1.3.6.1.4.1.2021.10.1.5.1 = INTEGER: 317
.1.3.6.1.4.1.2021.10.1.5.2 = INTEGER: 335
.1.3.6.1.4.1.2021.10.1.5.3 = INTEGER: 344
```
These values are taken from the NetSNMP maintained [UCD-SNMP-MIB.txt](https://www.net-snmp.org/docs/mibs/UCD-SNMP-MIB.txt) 

You can load this mib into the [ireasoning mibbrowser](https://www.ireasoning.com/mibbrowser.shtml) or view it online at the [online mib browser UCD-SNMP-MIB](https://mibbrowser.online/mibdb_search.php?mib=UCD-SNMP-MIB)

The vendor gives us a MIB description for these metrics as the following:

> The 1,5 and 15 minute load averages as an integer. This is computed by taking the floating point loadaverage value and multiplying by 100, then converting the value to an integer.

The SNMP agents identifies itself with the following system object ID:

```
snmpwalk -On -v2c -c public localhost:1610 .1.3.6.1.2.1.1.2
.1.3.6.1.2.1.1.2.0 = OID: .1.3.6.1.4.1.61509.42.1
```
Our objective is to create a configuration for OpenNMS which allows it to recognise devices with the system object ID `.1.3.6.1.4.1.61509.42.1`.
OpenNMS should then begin collecting the three load averages defined above from any device with that system object ID.

## Tasks

The lab for this exercise is in the [session5/minimal-minion-activemq](../session5/minimal-minion-activemq) directory using docker compose.

Example files and MIBs for this exercise are in [session5/example-configurations](../session5/example-configurations)

Before you spin up the minimal-minion-activemq, please verify with `docker ps` you don't have any other docker compose projects from previous sections running.

```bash
cd minimal-minion-activemq
docker compose up -d
```
You can access the following exposed services through the host system's localhost:

|URL/Command                           | Description                           |
|:-------------------------------------|:--------------------------------------|
|[http://localhost:8980](http://localhost:8980) | The OpenNMS Horizon web user interface (username:admin password:admin)|
|ssh&nbsp;admin@localhost&nbsp;-p&nbsp;8101 | The Horizon Karaf CLI via SSH<BR>If you dont have an SSH client, you can access this using `docker compose exec horizon ssh admin@localhost -p 8101` |
|ssh&nbsp;admin@localhost&nbsp;-p&nbsp;8201 | The Minion1 Karaf CLI via SSH<BR>`docker compose exec minion1 ssh admin@localhost -p 8201` |
|[http://localhost:3000](http://localhost:3000) |The Grafana web user interface|
|linux-01&nbsp;localhost:1610&nbsp;udp<BR>linux-02&nbsp;localhost:1611&nbsp;udp<BR>linux-02&nbsp;localhost:1612&nbsp;udp<BR>linux-04&nbsp;localhost:1613&nbsp;udp | Use these ports if you want to access the SNMP agents from your host system.<br>Inside the Docker `minimal-minion-activemq`, they are listening to the default port 161/udp.|

The docker compose project hosts the following containers:

```plain
                              Docker Compose Project
                             ┌──────────────────────────────────────────────────────────────────┐
External ports (on localhost)│ Internal addresses                                               │
                             │    ┌────────────────────┐              ┌────────────────────┐    │
                             │    │                    │              │                    │    │
                             │    │     database       │ N000         │      linux-01      │    │
                             │    │   172.20.0.10/24   ├──────┬───────│   172.20.0.101/24  ├────┼────── sysoId 61509 1610/udp
                             │    │                    │      │       │                    │    │
                             │    │                    │      │       │                    │    │
                             │    └────────────────────┘      │       └────────────────────┘    │
                             │                                │                                 │
                             │    ┌────────────────────┐      │       ┌────────────────────┐    │
                             │    │                    │      │       │                    │    │
       Web UI  8980/tcp ─────┼────┤      Horizon       │      │       │     linux-02       │    │
                             │    │   172.20.0.15/24   ├──────┼───────│   172.20.0.102/24  ├────┼────── sysoId 61509 1611/udp
  Karaf Shell  8101/tcp ─────┼────┤                    │      │       │                    │    │
                             │    │                    │      │       │                    │    │
                             │    └────────────────────┘      │       └────────────────────┘    │
                             │                                │                                 │
                             │    ┌────────────────────┐      │       ┌────────────────────┐    │
                             │    │                    │      │       │                    │    │
                             │    │      Grafana       │      │       │     linux-03       │    │
Grafana Web UI 3000/tcp ─────┼────│   172.20.0.26/24   ├──────┼───────│   172.20.0.103/24  ├────┼────── sysoId 8072 (Net-SNMP) 1612/udp
                             │    │                    │      │       │                    │    │
                             │    │                    │      │       │                    │    │
                             │    └────────────────────┘      │       └────────────────────┘    │
                             │                                │                                 │
                             │    ┌────────────────────┐      │       ┌────────────────────┐    │
                             │    │       Minion1      │      │       │                    │    │
                             │    │   172.20.0.25/24   ├──────┘       │     linux-04       │    │
  Karaf Shell  8201/tcp ─────┼────┤                    │              │                    ├────┼────── sysoId 8072 (Net-SNMP) 1613/udp
                             │    │                    │ N001         │                    │    │
                             │    │   172.20.2.25/24   ├──────────────┤   172.20.2.101/24  │    │
                             │    └────────────────────┘              └────────────────────┘    │
                             │                                                                  │
                             └──────────────────────────────────────────────────────────────────┘
```

### Hints

* Assign the role `ROLE_FILESYSTEM_EDITOR` to the `admin`, which allows you to edit configuration files through the web user interface. (See [Session 2 Modifying configuration files through UI](../session2/README.md#modifying-configuration-files-through-the-ui)
* If you want to start from scratch, run `docker compose down -v`, it will delete the database and the configuration files.
  You can start again with `docker compose up -d` afterwards.
* Ad-hoc data collection for a node from karaf consol : `opennms:collect -l Default -n linux-server:linux-03 org.opennms.netmgt.collectd.SnmpCollector linux-03`

### Task 1: Provision the four servers, linux-01, linux-02, linux-03 and linux-04

The provisioning requistion `linux-server` has the configurations for these devices.
You can load this from the `admin>Manage Provisioning Requisitions` page.

All the Linux servers have SNMP with v2c configured.
The read only community is set to `public`.

- [ ] Verify if you can get access to the SNMP agents sysoid using the Net-snmp `snmpwalk` command from inside one of the linux containers
  * `docker compose exec linux-03 snmpwalk -On -v2c -c public linux-02:161 .1.3.6.1.2.1.1.2`
- [ ] Verify if OpenNMS can access the SNMP agents using the karaf terminal snmp-walk command
  * `ssh admin@localhost -o UserKnownHostsFile=/dev/null -p 8101 snmp-walk -l Default linux-03 .1.3.6.1.4.1.2021.10.1.5` 

Question 1: How do the systems differentiate from each other regarding the SNMP information discovered?

Question 2: Investigate which metrics are collected from the servers, if not what do you think could be an issue?

Question 3: What are the steps you take to add the load average metrics to the configuration?

### Task 2: Create configuration to collect the Load average metrics

- [ ] Extend the default SNMP data collection with your own MIB OIDs and your system definition.
  * Create the configuration file `datacollection/exercise_5.1.xml`
  * Datacollection-group name: `exercise_5.1-group`
  * MIB object group name `<group name="exercise_5.1-loadavg"`
  * Create a system definition that matches the linux-01, linux-02 system object ID's
  * Include the `exercise_5.1-group` in the `default` snmp-collection in the `datacollection-config.xml`
- [ ] Verify if the metrics are collected as RRD files in the file system with:

```
docker compose exec horizon ls share/rrd/snmp/fs/linux-server/linux-01
```

### Task 3: Create RRD graph definition to visualise the metric

- [ ] Create a custom graph definition from the `snmp-graph.properties.d` directory
- [ ] Verify if you can see the graph in the Node Level Performance data
