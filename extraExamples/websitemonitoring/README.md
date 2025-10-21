## Web Site Monitoring example 

### introduction

This folder contains several examples of monitoring a popular web site from the USPTO.

An introductory video is provided here [Overview of OpenNMS web monitoring](https://youtu.be/1PDJjAS2NrU)

A more datailed video explains this example of monitoring a complex web site using OpenNMS. [Web Site Monitoring with OpenNMS](https://youtu.be/QcjqNDStjfw) 


The monitoring examples use 

[opennms metadata](https://docs.opennms.com/horizon/31/operation/deep-dive/meta-data.html) 

with the 

[HTTPPostMonitor](https://docs.opennms.com/horizon/30/reference/service-assurance/monitors/HttpPostMonitor.html)

and the

[PageSequenceMonitor](https://docs.opennms.com/horizon/30/reference/service-assurance/monitors/PageSequenceMonitor.html)

The use of metadata is based on notes in discourse [https://opennms.discourse.group/t/how-to-monitor-websites-using-metadata/1227](https://opennms.discourse.group/t/how-to-monitor-websites-using-metadata/1227)
but some corrections have been made.

The examples are rendered in OpenNMS as separate monitored services attached to IP addresses on a single node. 
The IP addresses have different services attached

![image](../websitemonitoring/images/OnmsUSPTOServices.png)


## USPTO monitoring using HttpPostMonitor

The USPTO patent search page is available here  https://ppubs.uspto.gov/pubwebapp/static/pages/ppubsbasic.html

![image](../websitemonitoring/images/PPubsSearch.png)

### OpenNMS service definition

![image](../websitemonitoring/images/WebPostusptoq1.png)

### description

This uses javascript to create a json based search 

POST https://ppubs.uspto.gov/dirsearch-public/searches/generic

Request body:

```
{
	"cursorMarker": "*",
	"databaseFilters": [
		{
			"databaseName": "USPAT"
		},
		{
			"databaseName": "US-PGPUB"
		},
		{
			"databaseName": "USOCR"
		}
	],
	"fields": [
		"documentId",
		"patentNumber",
		"title",
		"datePublished",
		"inventors",
		"pageCount"
	],
	"op": "OR",
	"pageSize": 50,
	"q": "(11521175).pn.",
	"searchType": 0,
	"sort": "date_publ desc"
}

```

The response to this search is 

```
{
	"cursorMarker": "AoJwgO6u8oQDPDEwMDAwMDY3MjgzOTYhVVMtVVMtMTE1MjExNzU=",
	"numFound": 1,
	"docs": [
		{
			"documentId": "US-11521175-B2",
			"datePublished": "2022-12-06",
			"title": "Patient sensor data exchange systems and methods",
			"patentNumber": "11521175",
			"inventors": "Dyell; David et al.",
			"pageCount": 16
		}
	]
}

```

In the OpenNMS test we use the [HTTPPostMonitor](https://docs.opennms.com/horizon/30/reference/service-assurance/monitors/HttpPostMonitor.html) to POST a json search string as a body in a request request for a NantHealth patent and receive back the patent references.

### configuration

in poller-configuration.xml

```
     <service name="Web-PostMonitor" interval="300000" user-defined="false" status="on">
         <pattern><![CDATA[^WebPost-.*$]]></pattern> 
         <parameter key="retry" value="1" />
         <parameter key="timeout" value="3000" />
         <parameter key="port" value="${requisition:port|443}" />
         <parameter key="scheme" value="${requisition:scheme|https}" />  
         <parameter key="ds-name" value="${service:name}" /> 
         <parameter key="rrd-repository" value="/opt/opennms/share/rrd/response"/>
         <parameter key="rrd-base-name" value="http-${requisition:rrd-base-name}" /> 
         <parameter key="uri" value="${requisition:path|/}" /> 
         <!-- response regex -->
         <parameter key="banner" value="${requisition:response-text}"/>
         <parameter key="payload" value="${requisition:payload}"/>
         <!-- header must have a value if used -->
         <parameter key="header0" value="Host:${requisition:vhost|localhost}"/>
         <parameter key="header1" value="Accept:${requisition:Accept|application/json}"/>
         <parameter key="header2" value="Content-Type:${requisition:Content-Type|application/json}"/>
         <parameter key="usesslfilter" value="${requisition:use-ssl-filter|false}" /> 
      </service>
      
      ...
      
     <monitor service="Web-PostMonitor" class-name="org.opennms.netmgt.poller.monitors.HttpPostMonitor"/>
       
```

The WebsitesUspto requisition defines a monitored service WebPost-usptoq1 which matches against the Web-PostMonitor definition above

```
<?xml version="1.0" encoding="UTF-8"?>
<model-import xmlns="http://xmlns.opennms.org/xsd/config/model-import" date-stamp="2022-12-16T06:43:49.732-05:00" foreign-source="WebsitesUspto"
   last-import="2022-12-16T06:44:44.866-05:00">
 
 <node foreign-id="web04-srv" node-label="web04-srv">
      <interface ip-addr="52.202.75.138" status="1" snmp-primary="N">
         <monitored-service service-name="WebPost-usptoq1">
            <meta-data context="requisition" key="vhost" value="ppubs.uspto.gov" />
            <meta-data context="requisition" key="path" value="/dirsearch-public/searches/generic" />
            <meta-data context="requisition" key="rrd-base-name" value="usptoq1" />
            <meta-data context="requisition" key="response" value="100-399" />
            <meta-data context="requisition" key="scheme" value="https" />
            <meta-data context="requisition" key="use-ssl-filter" value="true" />
            <meta-data context="requisition" key="port" value="443" />
            <meta-data context="requisition" key="response-text" value="US-11521175-B2" />
            <meta-data context="requisition" key="payload" value="{&quot;cursorMarker&quot;:&quot;*&quot;,&quot;databaseFilters&quot;:[{&quot;databaseName&quot;:&quot;USPAT&quot;},{&quot;databaseName&quot;:&quot;US-PGPUB&quot;},{&quot;databaseName&quot;:&quot;USOCR&quot;}],&quot;fields&quot;:[&quot;documentId&quot;,&quot;patentNumber&quot;,&quot;title&quot;,&quot;datePublished&quot;,&quot;inventors&quot;,&quot;pageCount&quot;],&quot;op&quot;:&quot;OR&quot;,&quot;pageSize&quot;:50,&quot;q&quot;:&quot;(11521175).pn.&quot;,&quot;searchType&quot;:0,&quot;sort&quot;:&quot;date_publ desc&quot;}" />
            <meta-data context="requisition" key="Accept" value="application/json"/>
            <meta-data context="requisition" key="Content-Type" value="application/json"/>
         </monitored-service>
<!--          <monitored-service service-name="ICMP" /> -->
<!--     <monitored-service service-name="HTTPS" /> -->
      </interface>
      
      <!-- this monitors the trade mark search service defined with the PageSequenceMonitor -->
      <!-- only ipv6 reponse to ping 151.207.240.78 -->
      <interface ip-addr="151.207.240.78" status="1" snmp-primary="N">
         <monitored-service service-name="UsptoTradeSearch">
         </monitored-service>
      </interface>
</node>


</model-import>

```

## USPTO trade mark search monitoring using PageSequenceMonitor

The USPTO Trade Mark search service is available here https://tmsearch.uspto.gov/bin/gate.exe?f=login&p_lang=english&p_d=trmk

![image](../websitemonitoring/images/TESSmainSearch1.png)

This offers a choice of search pages but the simplest one is the Basic Word Mark Search (New User) which selects this url

https://tmsearch.uspto.gov/bin/gate.exe?f=searchss&state=4805:2kzn4v.1.1

Note that the url perameter state (state=4805:2kzn4v.1.1) is the session key which is unique to the user session and must be included in all search requests and the final logout.

The Basic Word Mark Search page looks like this

![image](../websitemonitoring/images/TESSmainSearch2.png)

And a search result page looks like this

![image](../websitemonitoring/images/TESSmainSearch3.png)


### OpenNMS service

The opennms service page is below

![image](../websitemonitoring/images/usptoTradeSearch.png)

### description

The page sequence monitor goes through the following steps to perform a search to proves the service every 5 minutes.

1. load the main  page and search for the state variable to use in further queries
2. load the simple search page using the state variable from above
3. perform a search for a known NantHealth patent and check we get the document id
4. log out from the service using the state variable

The service is defined in poller-configuration.xml

```
      
      <service name="UsptoTradeSearch" interval="300000" user-defined="false" status="on">
         <parameter key="retry" value="1"/>
         <parameter key="timeout" value="3000"/>
         <parameter key="rrd-repository" value="/opt/opennms/share/rrd/response"/>
         <parameter key="rrd-base-name" value="usptotrade"/>
         <parameter key="ds-name" value="usptotrade"/>
         <parameter key="page-sequence">
         
             <page-sequence xmlns="">
             
               <!-- go to tess landing page https://tmsearch.uspto.gov/bin/gate.exe?f=login&p_lang=english&p_d=trmk -->
               <!-- and scrape the state reference from the <a href="/bin/gate.exe?f=searchss&state=4809:72vxz5.1.1">Basic Word Mark Search (New User)</a> tag -->
               <!-- using regex <a\s+(?:[^>]*?\s+)?href=(["'])(.*?)state=([^&"]+) -->
               <page disable-ssl-verification="true" host="${ipaddr}" 
               virtual-host="tmsearch.uspto.gov"
               http-version="1.1" method="GET" 
               path="/bin/gate.exe" 
               port="443" 
               scheme="https"
               response-range="100-399"
               successMatch="&lt;a\s+(?:[^>]*?\s+)?href=([&quot;'])(.*?)state=([^&amp;&quot;]+)" xmlns="">
               <parameter key="f" value="login" />
               <parameter key="p_lang" value="english" />
               <parameter key="p_d" value="trmk" />
               <session-variable name="state" match-group="3" />
               </page>
               
               <!-- start a search session page using the state variable extracted in last step -->
               <!-- https://tmsearch.uspto.gov/bin/gate.exe?f=searchss&state=4809:72vxz5.1.1 -->
               <page disable-ssl-verification="true" host="${ipaddr}" 
               virtual-host="tmsearch.uspto.gov"
               http-version="1.1" method="GET" 
               path="/bin/gate.exe" 
               port="443" 
               scheme="https"
               response-range="100-399"
               successMatch="&lt;I>New User" xmlns="">
                  <parameter key="f" value="searchss" />
                  <parameter key="state" value="${state}" />
               </page>
               
              <!-- Searching for OpenNMS trademark -->
              <!--  GET https://tmsearch.uspto.gov/bin/showfield?f=toc&state=4807:62zhxt.1.1&p_search=searchss&p_L=50&BackReference=&p_plural=yes&p_s_PARA1=&p_tagrepl~:=PARA1$LD&p_tagrepl~:=PARA2$COMB&expr=PARA1 AND PARA2&p_s_PARA2=OpenNMS&p_op_ALL=AND&a_default=search&a_search=Submit Query&a_search=Submit Query -->
              <!--  matches search for OpenNMS trademark against -->
                <page disable-ssl-verification="true" host="${ipaddr}" 
               virtual-host="tmsearch.uspto.gov"
               http-version="1.1" method="POST" 
               path="/bin/gate.exe" 
               port="443" 
               scheme="https"
               response-range="100-399"
               successMatch="87240338" xmlns="">
               
                  <parameter key="f" value="toc" />
                  <parameter key="state" value="${state}" />
                  <parameter key="p_search" value="search" /> 
                  <parameter key="p_s_All=" value="" />
                  <parameter key="p_s_All=" value="(opennms)[COMB]" />
                  <parameter key="a_default" value="search" />
                  <parameter key="a_search" value="Submit" />
               </page>

               
               <!-- Finally logout from this session-->
               <!--       <form method="POST" action="/bin/gate.exe"> -->
               <!--     <input type="hidden" name="state" value="4808:5ti3uh.1.1"> -->
               <!--     <input type="hidden" name="f" value="logout"> -->
               <!--       <input type="submit" name="a_logout" value="Logout"><i>Please logout when you are done to release system resources allocated for you.</i> -->
               <!-- </form> -->
  
               <page disable-ssl-verification="true" host="${ipaddr}" 
               virtual-host="tmsearch.uspto.gov"
               http-version="1.1" method="GET" 
               path="/bin/gate.exe" 
               port="443" 
               scheme="https"
               response-range="100-399"
               successMatch="Thank you for using US Trademark Electronic Search System" xmlns="">
                   <parameter key="f" value="logout" />
                   <parameter key="state" value="${state}" />
               </page>
               
            </page-sequence>
            
         </parameter>
      </service>
      
      ...
      
      <monitor service="UsptoTradeSearch" class-name="org.opennms.netmgt.poller.monitors.PageSequenceMonitor"/>
      
```
### running the examples

A docker-compose project is provided with all the required configurations. 

This can be run on a PC using [docker desktop](https://www.docker.com/products/docker-desktop/).

The example project runs containers for OpenNMS Horizon, Kafka, Postgres and 3 OpenNMS minions. 
(This example is based off other similar examples which you might find useful at [OpenNMS Forge stackplay](https://github.com/opennms-forge/stack-play) )

The example configurations for this tutorial are all held within the docker-compose project and are injected to the OpenNMS container when the example is run

The most important files for this demo are

[-opennms-home-/etc/poller-configuration.xml](../websitemonitoring/minimal-minion-kafka/container-fs/horizon/opt/opennms-overlay/etc/poller-configuration.xml)

and 

[-opennms-home-/etc/imports/WebsitesUspto.xml](../websitemonitoring/minimal-minion-kafka/container-fs/horizon/opt/opennms-overlay/etc/imports/WebsitesUspto.xml)


```
# to start the demo run
docker-compose up -d

#  the demo will take some time to come up but you can follow progress using
docker-compose logs -f horizon

# to end the demo run ( use -v only if you want to delete the database)
docker-compose down -v

# if you want to get inside the horizon container to look at for instance the /logs/poller.log
docker-compose exec horizon bash

```
once running you can see opennms at

https://localhost:8980

username admin
password admn

or if running ipv6

https://[::1]:8980

To start monitoring the USPTO web sites you will need to import the node definitions by synchronising the the WebsitesUspto requisition using the admin requisitions page at
http://[::1]:8980/opennms/admin/ng-requisitions/index.jsp#/requisitions

