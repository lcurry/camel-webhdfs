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
| ------------- |:-------------:| -----:|
| col 3 is      | right-aligned | $1600 |
| col 2 is      | centered      |   $12 |
| zebra stripes | are neat      |    $1 |




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
    
