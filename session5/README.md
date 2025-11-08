# Session 5 - Performance Data Collection

[Main Menu](../README.md) | [Session 5](../session5/README.md)

## Introduction

In [Session 2](../session3/README.md#service-and-web-site-monitoring) we saw how OpenNMS can be used to poll the availability of services on a regular basis using `synthetic transactions`.
The service monitoring function detects and polls services to ensure they are up.
It only stores the response time for each service request and events corresponding to the service life-cycle (discover, service up, service down etc).

In this session we will look at a more advanced OpenNMS [Performance Data Collection](https://docs.opennms.com/horizon/33/operation/deep-dive/performance-data-collection/introduction.html) capability which collects performance statistics on a regular interval from the service itself.

[performance-management-Introduction.pdf](../session5/performance-management-Introduction.pdf) Slides for this session.

You can examine published MIBS in an [Online MIM Browser https://mibbrowser.online/](https://mibbrowser.online/)

You can see which organisation is assigned a  sysOid enterprise number at [IANA assigned enterprise numbers](https://www.iana.org/assignments/enterprise-numbers/)

[Session 5 Video](https://youtu.be/f67ol8LD77s) (DEPRICATED - to be updated to match new material)

## Overview

The majority of performance data collection in SNMP deployments is performed using SNMP but many other performance data protocols are supported by OpenNMS including  `plain text/CSV`, `ReST/XML/Json`, `Java JMX` and `WBEM`. 

For more details see the [Performance Data Collectors Reference](https://docs.opennms.com/horizon/33/reference/performance-data-collection/introduction.html). 

In this session we will concentrate on SNMP data collection.

The key files configuring SNMP data collection are in the following list.

| Config File | Description |
|:------------|-------------|
| [etc/snmp-config.xml](../../main/pristine-opennms-config-files/etc-pristine/snmp-config.xml)  | Configures SNMP (version, community strings etc) per device.<BR>We covered SNMP configuration using the web ui in [Exercise-2-1 Adding an SNMP Community String](../session2/Exercise-2-1.md#adding-an-snmp-community-string)<BR>See also the example configurations for SNMP proxys and SNMP v3 in [etc/examples/snmp-config.xml](../../main/pristine-opennms-config-files/etc-pristine/examples/snmp-config.xml) |
| [etc/collectd-configuration.xml](../../main/pristine-opennms-config-files/etc-pristine/collectd-configuration.xml)|This file configures `collectd` to set up multiple polling packages that define filters to determine which IP addresses or node categories to collect each package from.<br>Packages can contain more than one protocol for data collection e.g SNMP, JMX, XML. Each protocol definition must reference the java class which is used to perform the collection. Each of these classes/protocols will have a different set of configuration files. |
| [etc/datacollection-config.xml](../../main/pristine-opennms-config-files/etc-pristine/datacollection-config.xml)| This file provides the configuration for SNMP data collection. It references files in the [etc/datacollection  folder](../../main/pristine-opennms-config-files/etc-pristine/datacollection/). For instance see the file [etc/datacollection/netsnmp.xml](../../main/pristine-opennms-config-files/etc-pristine/datacollection/netsnmp.xml) |
| [etc/xml-datacollection-config.xml](../../main/pristine-opennms-config-files/etc-pristine/xml-datacollection-config.xml)| This file is provided for information. It is not SNMP related. It provides the configuration for XML and Json data collection. It references files in the [etc/xml-datacollection  folder](../../main/pristine-opennms-config-files/etc-pristine/xml-datacollection).<BR>A similar pattern exists for other data collection protocols such as JMX |

The relationship between the SNMP data collection configuration files is illustrated below:

![alt text](../session5/images/collectd-config.png "Figure collectd-config.png")


## Exercises

[Exercise-5-1](../session5/Exercise-5-1.md)  Accessing the example network.

[Exercise-5-2](../session5/Exercise-5-2.md)  Collecting from an SNMP table.

[Exercise-5-3](../session5/Exercise-5-3.md)  Compiling a MIB.

[AdditionalNotes1.md](../session5/AdditionalNotes1.md) Miscellaneous notes.

