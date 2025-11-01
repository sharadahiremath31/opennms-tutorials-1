# Session 5 - Performance

[Main Menu](../README.md) | [Session 5](../session5/README.md)

[performance-management-Introduction.pdf](../session5/performance-management-Introduction.pdf) Slides for this session.

[Session 5 Video](https://youtu.be/f67ol8LD77s) (DEPRICATED - to be replaced)

[Exercise-5-1](../session5/Exercise-5-1.md)

[Exercise-5-2](../session5/Exercise-5-2.md)

[Exercise-5-3](../session5/Exercise-5-3.md)

[AdditionalNotes1.md](../session5/AdditionalNotes1.md)

## Exercise 5.1

We have an SNMP enabled device which provides the system load as a metric to indicate the system CPU and I/O utilization.
The metric is provided under the following SNMP object ids:

```
snmpwalk -On -v2c -c public localhost:1610 .1.3.6.1.4.1.2021.10.1.5
.1.3.6.1.4.1.2021.10.1.5.1 = INTEGER: 317
.1.3.6.1.4.1.2021.10.1.5.2 = INTEGER: 335
.1.3.6.1.4.1.2021.10.1.5.3 = INTEGER: 344
```
These values are taken from the NetSNMP maintained [UCD-SNMP-MIB.txt](https://www.net-snmp.org/docs/mibs/UCD-SNMP-MIB.txt) 

The vendor gives us a MIB description for these metrics as the following:

> The 1,5 and 15 minute load averages as an integer. This is computed by taking the floating point loadaverage value and multiplying by 100, then converting the value to an integer.

The SNMP agents identifies itself with the following system object ID:

```
snmpwalk -On -v2c -c public localhost:1610 .1.3.6.1.2.1.1.2
.1.3.6.1.2.1.1.2.0 = OID: .1.3.6.1.4.1.61509.42.1
```

The lab for this exercise is in the `minimal-minion-activemq` directory using docker compose.

Before you spin up the minimal-minion-activemq, please verify with `docker ps` you don't have any other docker compose projects from previous sections running.

```bash
cd minimal-minion-activemq
docker compose up -d
```
You can access the following exposed services through the host system's localhost:

|URL/Command                           | Description                           |
|:-------------------------------------|:--------------------------------------|
|http://localhost:8980                 | The OpenNMS Horizon web user interface|
|ssh&nbsp;admin@localhost&nbsp;-p&nbsp;8101 | The Horizon Karaf CLI via SSH |
|ssh&nbsp;admin@localhost&nbsp;-p&nbsp;8201 | The Minion1 Karaf CLI via SSH |
|http://localhost:3000                 |The Grafana web user interface|
|linux-01&nbsp;localhost:1610&nbsp;udp<BR>linux-02&nbsp;localhost:1611&nbsp;udp<BR>linux-02&nbsp;localhost:1612&nbsp;udp<BR>linux-04&nbsp;localhost:1613&nbsp;udp | Use these ports if you want to access the SNMP agents from your host system.<br>Inside the Docker minimal-minion-activemq, they are listening to the default port 161/udp.|



```plain
                              Docker Compose Project
                             ┌──────────────────────────────────────────────────────────────────┐
External ports (on localhost)│ Internal addresses                                               │
                             │    ┌────────────────────┐              ┌────────────────────┐    │
                             │    │                    │              │                    │    │
                             │    │     database       │ N000         │      linux-01      │    │
                             │    │   172.20.0.10/24   ├──────┬───────│   172.20.0.101/24  ├────┼────── Net-SNMP 1610/udp
                             │    │                    │      │       │                    │    │
                             │    │                    │      │       │                    │    │
                             │    └────────────────────┘      │       └────────────────────┘    │
                             │                                │                                 │
                             │    ┌────────────────────┐      │       ┌────────────────────┐    │
                             │    │                    │      │       │                    │    │
       Web UI  8980/tcp ─────┼────┤      Horizon       │      │       │     linux-02       │    │
                             │    │   172.20.0.15/24   ├──────┼───────│   172.20.0.102/24  ├────┼────── Net-SNMP 1611/udp
  Karaf Shell  8101/tcp ─────┼────┤                    │      │       │                    │    │
                             │    │                    │      │       │                    │    │
                             │    └────────────────────┘      │       └────────────────────┘    │
                             │                                │                                 │
                             │    ┌────────────────────┐      │       ┌────────────────────┐    │
                             │    │                    │      │       │                    │    │
                             │    │      Grafana       │      │       │     linux-03       │    │
Grafana Web UI 3000/tcp ─────┼────│   172.20.0.26/24   ├──────┼───────│   172.20.0.103/24  ├────┼────── Net-SNMP 1612/udp
                             │    │                    │      │       │                    │    │
                             │    │                    │      │       │                    │    │
                             │    └────────────────────┘      │       └────────────────────┘    │
                             │                                │                                 │
                             │    ┌────────────────────┐      │       ┌────────────────────┐    │
                             │    │       Minion1      │      │       │                    │    │
                             │    │   172.20.0.25/24   ├──────┘       │     linux-04       │    │
  Karaf Shell  8101/tcp ─────┼────┤                    │              │                    ├────┼────── Net-SNMP 1613/udp
                             │    │                    │ N001         │                    │    │
                             │    │   172.20.2.25/24   ├──────────────┤   172.20.2.101/24  │    │
                             │    └────────────────────┘              └────────────────────┘    │
                             │                                                                  │
                             └──────────────────────────────────────────────────────────────────┘
```

### Hints

* Assign the role `ROLE_FILESYSTEM_EDITOR` to the `admin`, which allows you to edit configuration files through the web user interface. (See [Session 2 Modifying configuration files through UI](../session2/README.md2#modifying-configuration-files-through-the-ui)
* If you want to start from scratch, run `docker compose down -v`, it will delete the database and the configuration files.
  You can start again with `docker compose up -d` afterwards.
* Ad-hoc data collection for a node: `opennms:collect -l Default -n 4 org.opennms.netmgt.collectd.SnmpCollector 192.168.42.34`
* [PGtune](https://pgtune.leopard.in.ua) tool for generating a configuration in PostgreSQL


### Task 1: Provision the three servers, linux-01, linux-02 and linux-03

All three Linux server have SNMP with v2c configured.
The read only community is set to `public`.
- [ ] Verify if you can get access to the SNMP agents using the `snmpwalk` command.
- [ ] Verify if OpenNMS can access the SNMP agents using `ssh admin@localhost -p 8101 snmp-walk -l Default 192.168.42.34 .1.3.6.1.4.1.2021.10.1.5` 

Question 1: How do the 3 systems differentiate from each other regarding the SNMP information discovered?

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
docker compose exec core ls share/rrd/snmp/fs/linux-server/linux-01
```

### Task 3: Create RRD graph definition to visualise the metric

- [ ] Create a custom graph definition from the `snmp-graph.properties.d` directory
- [ ] Verify if you can see the graph in the Node Level Performance data

## Requested topics from last session on Tuesday

### Question: How can we change the timezone formatting in the web user interface? The default is not very user friendly.
Answer: Set the `org.opennms.ui.datettimeformat` property

Link to Formatting instructions: [DataTimeFormatter](https://docs.oracle.com/javase/8/docs/api/java/time/format/DateTimeFormatter.html)
```
opennms@e45dfa7efd9c:~$ cat etc/opennms.properties.d/timeformat.properties
org.opennms.ui.datettimeformat=yyyy-MM-dd HH:mm:ss (z)
```

### Controlling data collections

- [ ] Limit nodes from SNMP data collection based on a surveillance category `Collect-Enabled`

File: `etc/collectd-configuration.xml`
```xml
<package name="example1" remote="false">
  <filter>categoryName == 'Collect-Enabled'</filter>
  <include-range begin="1.1.1.1" end="254.254.254.254"/>
  <include-range begin="::1" end="ffff:ffff:ffff:ffff:ffff:ffff:ffff:ffff"/>
  ...
```

- [ ] Assign surveillance category `Collect-Enabled` to `linux-01 and verify the collection packages in the web user interface all three linux nodes
- [ ] Change the data collection frequency and how they relate to collection packages and RRD/JRobin files
- [ ] Assign node `linux-03` to the surveillance category `Collect-10s`
- [ ] Create a new collection set `default-10s` in the datacollection-config.xml, RRDTool and JRobin limitations
- [ ] Add a second SNMP collection package with the collection interval 10 sec instead of 5 min.

File: `etc/collectd-configuration.xml`
```xml
<package name="collect-10s" remote="false">
  <filter>categoryName == 'Collect-10s'</filter>
  <include-range begin="1.1.1.1" end="254.254.254.254"/>
  <include-range begin="::1" end="ffff:ffff:ffff:ffff:ffff:ffff:ffff:ffff"/>
  <service name="SNMP" interval="10000" user-defined="false" status="on">
    <parameter key="collection" value="default-10s"/>
    <parameter key="thresholding-enabled" value="true"/>
  </service>
</package>
```

IMPORTANT: To apply the settings Collectd needs a reload when a category was changed in the inventory

- [ ] Reload Collectd configuration with `ssh admin@localhost -p 8101 reload-daemon Collectd`
- [ ] It seems like we have a bug in 32.0.5 with the filter, as soon one filter applies for a node both collection packages are applied

### Thresholding

- [ ] Create a high threshold for CPU usage higher than 50%

![cpu-user-threshold.png](images%2Fcpu-user-threshold.png)

- [ ] Install the `stress-ng` command on `linux-03` which can be used to simulate a high workload to trigger a threshold exceeded

```bash
docker compose exec -u root linux-03 apk add --no-cache stress-ng # Install the required package in the linux-03 container
docker compose exec -u root linux-03 stress-ng -c 1 -l 55 # Simulate ~55% CPU usage
```

### REST API calls for measurements

```
curl -u admin http://localhost:8980/opennms/rest/measurements/node%5B6%5D.nodeSnmp%5B%5D/CpuRawUser?start=-7200000&maxrows=30&aggregation=AVERAGE
```

REST API call examples can be used from `rest-examples` directory using the REST API tool [Bruno](https://www.usebruno.com).

## Exercise 5.2: Collecting from an SNMP Table

### Step 1: Create a resource type for entries in the table

![hr-description.png](images/hr-description.png)

File: `etc/resource-types.d/exercise_5.2.xml`

```xml
<?xml version="1.0"?>
<resource-types>
    <resourceType name="hrStorageIndex" label="Host Resources Storage" resourceLabel="${hrStorageDescr}">
        <persistenceSelectorStrategy class="org.opennms.netmgt.collection.support.PersistAllSelectorStrategy"/>
        <storageStrategy class="org.opennms.netmgt.dao.support.SiblingColumnStorageStrategy">
            <parameter key="sibling-column-name" value="hrStorageDescr"/>
            <parameter key="replace-first" value="s/^-$/_root_fs/"/>
            <parameter key="replace-all" value="s/^-//"/>
            <parameter key="replace-all" value="s/\s//"/>
            <parameter key="replace-all" value="s/:\\.*//"/>
        </storageStrategy>
    </resourceType>
</resource-types>
```

### Step 2: Create data-collection group

Define which metrics you want to collect on which SNMP agent, in our case identified by system object ID: `.1.3.6.1.4.1.61509.42.1`

![hr-entry.png](images/hr-entry.png)

File: `etc/datacollection/exercise_5.2.xml`

```xml
<datacollection-group xmlns="http://xmlns.opennms.org/xsd/config/datacollection" name="exercise_5.2-group">
  <group name="exercise_5.2-host-resources-storage" ifType="all">
    <mibObj oid=".1.3.6.1.2.1.25.2.3.1.2" instance="hrStorageIndex" alias="hrStorageType" type="string"/>
    <mibObj oid=".1.3.6.1.2.1.25.2.3.1.3" instance="hrStorageIndex" alias="hrStorageDescr" type="string"/>
    <mibObj oid=".1.3.6.1.2.1.25.2.3.1.4" instance="hrStorageIndex" alias="hrStorageAllocUnits" type="gauge"/>
    <mibObj oid=".1.3.6.1.2.1.25.2.3.1.5" instance="hrStorageIndex" alias="hrStorageSize" type="gauge"/>
    <mibObj oid=".1.3.6.1.2.1.25.2.3.1.6" instance="hrStorageIndex" alias="hrStorageUsed" type="gauge"/>
  </group>
  <systemDef name="Exercise-5.1-SNMP-System">
    <sysoid>.1.3.6.1.4.1.61509.42.1</sysoid>
    <collect>
      <includeGroup>exercise_5.2-host-resources-storage</includeGroup>
    </collect>
  </systemDef>
</datacollection-group>

```

### Step 3: Associate a data collection group to an SNMP collection

File: `etc/datacollection-config.xml`

```xml
<snmp-collection name="exercise-5.2-snmp-collection" snmpStorageFlag="select">
  <rrd step="300">
    <rra>RRA:AVERAGE:0.5:1:20160</rra>
    <rra>RRA:AVERAGE:0.5:12:14880</rra>
    <rra>RRA:AVERAGE:0.5:288:3660</rra>
    <rra>RRA:MAX:0.5:288:3660</rra>
    <rra>RRA:MIN:0.5:288:3660</rra>
  </rrd>
  <include-collection dataCollectionGroup="exercise_5.2-group"/>
</snmp-collection>
```

### Step 4: Define an SNMP collection service with the SNMP collection

File: `etc/collectd-configuration.xml`

```xml
<package name="Exercise-5.2-SNMP-Package" remote="false">
  <filter>IPADDR != '0.0.0.0'</filter>
  <include-range begin="1.1.1.1" end="254.254.254.254"/>
  <include-range begin="::1" end="ffff:ffff:ffff:ffff:ffff:ffff:ffff:ffff"/>
  <service name="SNMP-Custom-Agent" interval="30000" user-defined="false" status="on">
    <parameter key="collection" value="exercise-5.2-snmp-collection"/>
    <parameter key="thresholding-enabled" value="true"/>
  </service>
</package>
```

### Step 5: Verify data collection

```
docker compose exec core ls share/rrd/snmp/fs/linux-server/linux-01
```

## Exercise 5.3:  Compiling a MIB for data collection

- [ ] Upload the MIBS from the `mibs` directory into the MIB compiler using the web interface, compile the MIBs in the correct dependency order
  - SNMPv2-SMI.mib
  - SNMPv2-TC.mib
  - UCD-SNMP-MIB.mib
- [ ] Generate a data collection configuration from the UCD-SNMP-MIB
- [ ] Investigate and discuss the generated configuration files and graph defintions (`datacollection/UCD-SNMP-MIB.xml` and `snmp-graph.properties.d/UCD-SNMP-MIB.properties`)
