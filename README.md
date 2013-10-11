WEBHDFS Component
=============

A camel component that uses the WEBHDFS API to communicate with Hadoop.
This component enables you to write messages into an HDFS file system.
WEBHDFS is the RESTful API into Hadoop. 

Maven users will need to add the following dependency to their pom.xml for this component:

```xml
		<dependency>
			<groupId>org.apache.camel</groupId>
			<artifactId>camel-webhdfs</artifactId>
			<version>x.x.x</version>
			 <!-- use the same version as your Camel core version -->
		</dependency>
```


URI Format
==========

    > webhdfs://hostname[:port][/path][?options]

You can append query options to the URI in the following format, ?option=value&option=value&...   


Options
=======

| Name        | Default Value           | Description  |
| ------------- |-------------|-----|
| path      | Null | An expression that specifies a sub-directory (or directories) to wite files into HDFS. The location written to in HDFS is determined by: the primary path (specified before the query options in the URI) + the value specified for the path option. The value of path option is appended onto the end of the primary path.|
| key      | Null      | An expression that specifies the correlation key. The value of the expression determines which "correlation group" a message belongs to. All messages sent with the same key will be aggregated into same file.  |
| aggregationSize | 128000 (128KB)      | In bytes, size of all aggregated messages for a given message group. This will determine the cumulative size of files written to HDFS. |
| aggregationTimeout | 300000 (5 mins)     | In ms, All messages of the same message group are appended to the same file. Whenever the  aggregationTimeout expires for a message group, a new file is created to hold subsequent messages associated with the group. The aggregationTimeout is measured as idle time (no messages arrive) for a message group.|


Usage
=====
The WEBHDFS component supports the producer endpoint only. 

Producer Example
================
The following is an example of how to send messages to HDFS via camel WEBHDFS component.

In Java DSL

Or in Spring XML

```xml
<route>
    <from uri="activemq:queue:foo"/>
    <to uri="webhdfs://localhost:50070/webhdfs/v1/user/fuse?path=${header.path}&amp;key=${header.key}"/>
<route>
```


The implementation of the Camel WEBHDFS component leverages many features of the Aggregator pattern. 
The aggregator is a generic component offered by Apache Camel. More information is available on the Camel Aggregator here:
http://camel.apache.org/aggregator2.html


Message Groups
==============
Each message that enters the webhdfs endpoint is assigned to a message group. This assignment is based on a 
correlation identifier (key.) The correlation key determines which message group a message belongs to. 

* key: The correlation key designates which message group a message belongs.

A route could be written in such a way that the key option of the webhdfs endpoint is derived
from a header of the incoming message. That way, clients sending the messages can explicitely specify the key.
For example, the key could be specified by client as a query string paramter in an incoming HTTP message, or
as a JMS message header.
With the appropriate expression as the value of the key option for the webhdfs endpoint 
a client can specify the key and thus be sure that all his/her messages sent with the same key go to the same file(s). 

The component will make sure the messages with the same key 
get written to the same file. If a key is not specified, then the component assigns a key based on reasonable 
defaults. 

Completion Conditions
=====================
In general, all messages that belong to the same message group will get written to the same file. 
There are, however, limits on the number of messages that will be written to a single file. It would 
not make sense to write all messages to the same file because that file would grow too large. The 
completion condition will determine how files get broken up into reasonable size files that make sense 
for your environment. The values on these limits are configurable. You can control the frequency 
and size of files that the component writes into Hadoop by configuring values for completion conditions.  
The component will continue to append all messages belonging to the same message group, until either one of the 
following two completion conditions occur.
aggregationTimeout – This configures the amount of time, in milliseconds, that a message group will remain active. The time measures idle time (no messages arrive.) Whenever a message arrives for a given message group, the timer starts over for that particular message group. If the timeout ever expires for a message group, the file will be written and no further messages will be appended to that file. 
aggregationSize – The component keeps track of the total size of all messages sent to it for a given message group. Once this configured size is reached, the condition will trigger and the final message is written, and no further messages will be appended to that file.
As soon as one of the above limits is met for a given message group, the completion condition will fire. The message that caused the completion condition to trigger will be the last message written to the file. At that point, the file associated with the message group will be marked as done and no more messages are written to that file. Subsequent messages will be assigned to a new message group, and written to a new file.



There are three main build targets associated with corresponding maven profiles

* fab: Fuse Fabric
* amq: Fuse A-MQ
* esb: Fuse ESB
* release: All of the above


Building JBOSS Fuse
===================
There are three main build targets associated with corresponding maven profiles

* fab: Fuse Fabric
* amq: Fuse A-MQ
* esb: Fuse ESB
* release: All of the above

Examples
--------

Build Fuse Fabric and run the associated smoke tests

    > mvn clean install
    
Build Fuse A-MQ and run the associated tests

    > mvn -Pamq clean install
    
Build Fuse ESB and run the associated tests

    > mvn -Pesb clean install
    
Build all modules and run the associated smoke tests

    > mvn -Prelease clean install

Note, to avoid getting prompted for a gpg key add **-Dgpg.skip=true**

If you want to build everything without running tests and get right to running fabric with a default user

    > mvn clean install -Dtest=false -Dgpg.skip=true -Pdev,release
    
Test Profiles
-------------

Fuse Fabric tests are seperated in serveral dedicated tests profiles

* ts.smoke:   Smoke tests
* ts.basic:   Basic integration tests
* ts.wildfly: WildFly integration tests
* ts.all:     All of the above

Examples
--------

Build Fuse Fabric and run the smoke and basic integration tests

    > mvn -Dts.basic clean install
    
Build Fuse Fabric and run all tests

    > mvn -Dts.all clean install
    
Build all modules and run all tests

    > mvn -Prelease -Dts.all clean install
    
Build Fuse Fabric and skip the smoke tests

    > mvn -Dts.skip.smoke clean install
    
