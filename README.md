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
| path      | Null | The path option specifies a sub-directory (or directories) to wite files into HDFS. The location written to in HDFS is determined by: the primary path (specified before the query options in the URI) + the value specified for the path option. The value of path option is appended onto the end of the primary path.|
| key      | Null      | An expression that specifies the correlation key. The value of the expression determines which "correlation group" a message belongs to. All messages sent with the same key will be aggregated into same file.  |
| aggregationSize | 128000 (128KB)      | In bytes, size of all aggregated messages for a given message group. This will determine the cumulative size of files written to HDFS. |
| aggregationTimeout | 300000 (5 mins)     | In ms, All messages of the same message group are appended to the same file. Whenever the  aggregationTimeout expires for a message group, a new file is created to hold subsequent messages associated with the group. The aggregationTimeout is measured as idle time (no messages arrive) for a message group.|




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
    
