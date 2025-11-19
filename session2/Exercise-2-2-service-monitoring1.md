[Main Menu](../README.md) | [Session 2](../session2/README.md) | [Exercise-2-2-service-monitoring](../session2/Exercise-2-2-service-monitoring1.md)

# Exercise Service Monitoring

# Introduction
OpenNMS has separate service polling and data collection frameworks. 

If data colection is defined for a service, OpenNMS will attempt to collect data (e.g. SNMP data such as bits in/out) for a node or interface.
We will cover data collection in a later session.

Service polling does not do data collection but does perform simple synthetic transactions against all the defined services on each interface at regular intervals (Usually every 5 minutes). 
Polling determines whether the service is up or down and measures the response time for each service.

The services which can be polled are defined in the file [/etc/poller-configuration.xml](../pristine-opennms-config-files/etc-pristine//poller-configuration.xml)

in `poller-configuration.xml` you will see various packages defined.
A package defines a group of services for regular polling. 
It determines how often the services within a package will be polled and how this data will be aggregated in `rrd files` if they are being used.

A filter definition within a package can also determine which nodes are eligable to be included in the package depending on their IP address range and criteria such as their `category`.

Each package may also have a set of `downtime` definitions. 
The OpenNMS downtime model determines wht OpenNMS will do if a service fails to repond to a poll. 
Typically, OpenNMS will speed up the polling rate for a period to more accurately detect when the service comes up so that SLA's are measured more accurately. 
However, after this period, OpenNMS will slow the polling right down so that missing services dont impact the polling of other services and it is also possible to  configure OpenNMS to delete a servcie if it dissapears for a long time.

At the bottom of the file, you will see that each service definition also has an assocaited monitor which defines the java class OpenNMS uses to poll a particular service definition

In the pristine [/etc/poller-configuration.xml](../pristine-opennms-config-files/etc-pristine//poller-configuration.xml) you will see an `example1` package definition which is the 'out of the box' definition for polling most services.

Specifically we will look at the 'HTTPS' service definition in the `example1` package.

```
   <package name="example1">
      <filter>IPADDR != '0.0.0.0'</filter>
      <include-range begin="1.1.1.1" end="254.254.254.254" />
      <include-range begin="::1" end="ffff:ffff:ffff:ffff:ffff:ffff:ffff:ffff" />
      <rrd step="300">
         <rra>RRA:AVERAGE:0.5:1:2016</rra>
         <rra>RRA:AVERAGE:0.5:12:1488</rra>
         <rra>RRA:AVERAGE:0.5:288:366</rra>
         <rra>RRA:MAX:0.5:288:366</rra>
         <rra>RRA:MIN:0.5:288:366</rra>
      </rrd>
---
      <service name="HTTPS" interval="300000" user-defined="false" status="on">
         <parameter key="retry" value="${requisition:poller-retry|requisition:retry|detector:retry|1}" />
         <parameter key="timeout" value="${requisition:poller-timeout|requisition:timeout|detector:timeout|5000}" />
         <parameter key="port" value="${requisition:port|detector:port|443}" />
         <parameter key="url" value="${requisition:url|detector:url|/}" />
         <parameter key="rrd-repository" value="/opt/opennms/share/rrd/response" />
         <parameter key="rrd-base-name" value="https" />
         <parameter key="ds-name" value="https" />
      </service>
---
      <downtime begin="0" end="300000" interval="30000" /><!-- 30s, 0, 5m -->
      <downtime begin="300000" end="43200000" interval="300000" /><!-- 5m, 5m, 12h -->
      <downtime begin="43200000" end="432000000" interval="600000" /><!-- 10m, 12h, 5d -->
      <downtime begin="432000000" interval="3600000" /><!-- 1h, 5d -->
   </package>

---

   <monitor service="HTTPS" class-name="org.opennms.netmgt.poller.monitors.HttpsMonitor" />

```

You will note that the HTTPS service is monitored using the HttpsMonitor which requires a number of paramaters to be defined. 
These paramaters can be altered on a per node basis using paramater subsctitution from matadata defined for the node.

For instance, the HTTPS port will default to 443 but can be changed by a metadata name/value pair from the requisition metadata.

```
<parameter key="port" value="${requisition:port|detector:port|443}" />
```

In our example, we will change the url which is polled by adding the `/wordpress/` page to the HTTPS service definition.
```
         <parameter key="url" value="${requisition:url|detector:url|/}" />
```

## service 3 tier network

Often sites are made scalable and more resilient through load-balancing a number of servers. 
In this example we have installed three wordpress servers behind an NGINX load balancer.

All the servers share the same MariaDB database which also maintains the user session data across the servers.

![alt text](../session2/images/examplenetwork-loadbalance.drawio.png "Figure examplenetwork-loadbalance.drawio.png")

You can see the wordpress configuration if you look at the [docker-compose.yml](../session2/minimal-minion-activemq/docker-compose.yaml) file.
This contains a section for the OpenNMS containers, a section for three netsnmp containers (for a separate set of exercises) and at the bottom a section for the Wordpress containers.

You will see three Wordpress containers; wordpress1, wordpress2  and wordpress3. 
These all share the same data volume `wp_data` so that they are effectively clones of each other.

You will also see a wordpress-cli container. 
This container waits for 20 seconds after the project starts to allow the database and the other containers to start before initialising the word press containers, giving them their admin username and password and an initial front page name for the site using the command:

```
      /bin/sh -c '
      sleep 20;
      wp core install --path="/var/www/html/wordpress" --url="https://localhost/wordpress" --title="Local Wordpress By Docker" --admin_user=admin --admin_password=secret --admin_email=foo@bar.com
      '
```
After this initialisation step, this container exits.
Without wordpress-cli, you would have to manually configure the Wordpress instance on first start.

The nginx configuration is held in [wp-load-balencer-with-ssl.conf](..\session2\minimal-minion-activemq\container-fs\nginx\conf.d\wp-load-balencer-with-ssl.conf).
This sets up https termination and the round robin request forwarding to the three Wordpress servers.

## running the example

```
cd minimal-minion-activemq
docker compose up -d

[+] Running 21/21
 ✔ Network minimal-minion-activemq_N001               Created
 ✔ Network minimal-minion-activemq_N000               Created
 ✔ Volume "minimal-minion-activemq_wp_data"           Created
 ✔ Volume "minimal-minion-activemq_db_data"           Created
 ✔ Volume "minimal-minion-activemq_data-opennms"      Created
 ✔ Volume "minimal-minion-activemq_data-opennms-cfg"  Created
 ✔ Volume "minimal-minion-activemq_data-postgres"     Created

OpenNMS containers
 ✔ Container database                                 Started
 ✔ Container horizon                                  Started
 ✔ Container minion1                                  Started
 
SNMP Example Containers
 ✔ Container netsnmp_1_1                              Started
 ✔ Container netsnmp_1_2                              Started
 ✔ Container netsnmp_2_1                              Started
 ✔ Container netsnmp_2_2                              Started
 ✔ Container chubb_camera_01                          Started

Service Management Example  containers ( Wordpress Service)
 ✔ Container nginx                                    Started
 ✔ Container db                                       Started
 ✔ Container wordpress1                               Started
 ✔ Container wordpress2                               Started
 ✔ Container wordpress3                               Started
 ✔ Container wordpress-cli                            Started
```

## Testing Word press and the load balancer

The three wordpress instances share the same configuration and static data.
They are also connected to the same database.

The `wordpress-cli` container configures the wordpress instances with an initial user and an initial front page blog post.

All three containers sit behind the nginx container which both terminates the https/tls traffic and acts as a round robin load balancer.

browse to `https://localhost` and view the following landing page with links to wordpress and wordpress admin pages

![alt text](../session2/images/nginx-html.png "Figure nginx-html.png")

the main wordpress page at [https://localhost/wordpress/](https://localhost/wordpress/) is illustrated below.

![alt text](../session2/images/wordpress1.png "Figure wordpress1.png")

The wordpress admin page is at [https://localhost/wordpress/wp-login.php](https://localhost/wordpress/wp-login.php) and this allows login with a username and password 
* Username admin
* Password secret

![alt text](../session2/images/wordpress2.png "Figure wordpress2.png")

The wordpress admin page should show up as below.

![alt text](../session2/images/wordpress3.png "Figure wordpress3.png")

## Testing the Round-Robin load balancing 

To test that the load balancing is working, open three terminals use each one to view the word press logs for each of the wordpress instances

```
docker compose logs -f wordpress1

docker compose logs -f wordpress2

docker compose logs -f wordpress3
```

If you repeatedly refresh the Wordpress login page, you should see the logs advance one at a time for each of the wordpress instances as they are polled by the load balancer.

![alt text](../session2/images/loadbalance-example1.png "Figure loadbalance-example1.png")


## Monitoring with OpenNMS

OpenNMS can monitor the load balancer, each of the wordpress servers and the state of the database.

browse to [http://localhost:8980/opennms](http://localhost:8980/opennms) and login using 
* user:admin 
* password:admin

Skip the change password dialog.

Go to the requisitions page and inspect the test-wordpress requisition.

![alt text](../session2/images/wordpressRequisition1.png "Figure wordpressReqisition1.png")

This UI is backed by the requisition import file [test-wordpress.xml](../session2/minimal-minion-activemq/container-fs/horizon/opt/opennms-overlay/etc/imports/test-wordpress.xml)

Open the wordpress-load-balancer entry for editing. (Use the Vertical Layout).

![alt text](../session2/images/wordpressRequisition2.png "Figure wordpressReqisition2.png")

Note the Meta-data entry which is a `service` scoped name value pair 
* name : url
* value: /wordpress/
* scope: service HTTPS

![alt text](../session2/images/metadata1.png "Figure metadata1.png")


This corresponds to the service definition for the HTTPS service on the load balancer node which means we are specifying in the requisition the specific URL to be tested for the wordpress load balancer service.


![alt text](../session2/images/https-loadbalanceService1.png "Figure https-loadbalanceService1.png")


Synchronise this requisition and see each of the servers appear in the nodes page
![alt text](../session2/images/wpnodes1.png "Figure wpnodes1.png")

## simulate a node failure

Try stopping one of the servers and see what happens

```
docker compose stop wordpress1
```
After a polling period, you should see service failures on the front page.

![alt text](../session2/images/wpfailFront1.png "Figure wpfailFront1.png")

## Business Service Monitoring

It is possible to combine service outages using the Business Service Monitoring feature as shown below.

![alt text](../session2/images/https-loadbalanceServiceTopology.png "Figure https-loadbalanceServiceTopology.png")

For more information see [Business Service Monitoring](https://docs.opennms.com/horizon/33/operation/bsm/introduction.html)
More details of configuring the graph of business services will be covered in a later session. 




