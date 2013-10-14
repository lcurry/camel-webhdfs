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

The implementation of the camel webhdfs component utilizes the
[HTTP REST API interface into HDFS](http://hadoop.apache.org/docs/r2.1.0-beta/hadoop-project-dist/hadoop-hdfs/WebHDFS.html).


URI Format
==========

    > webhdfs://hostname[:port][/path][?options]

You can append query options to the URI in the following format, ?option=value&option=value&...   

Options
=======

| Name        | Default Value           | Description  |
| ------------- |-------------|-----|
| default.data.dir      | /user/fuse | Specifies default location to wite files into HDFS. |
| path      | Null | An expression that specifies a sub-directory (or directories), relative to the 'default.data.dir' to wite files into HDFS. The ultimate location where the file is written is determined by appending the value of this 'path' option (if specified) onto the end of the 'default.data.dir'.|
| key      | Null      | An expression that specifies the correlation key. The value of the expression determines which correlation group a message belongs to. All messages sent with the same key will be aggregated into same file, up to a designated completion condition. See options aggregationSize and aggregationTimeout below for specifying competion conditions.  |
| aggregationSize | 128000000 (128MB)      | In bytes, total size of all aggregated messages for a given message group. The message that causes this number of bytes received to be exceeded for a message group, will cause the completion condition to fire and will be the last message to be written to the file. This option will determine the cumulative size of files written to HDFS. |
| aggregationTimeout | 300000 (5 mins)     | In ms, All messages of the same message group are appended to the same file. Whenever the  aggregationTimeout expires for a message group, a new file is created to hold subsequent messages associated with the group. The aggregationTimeout is measured as idle time (no messages arrive) for a message group.|
| fileExtension | txt     | File extension of all files written to hdfs. |
| username | fuse     | The username the component will use for webhdfs HTTP calls to Hadoop server. |

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
    <to uri="webhdfs://localhost:50070/webhdfs/v1?path=${header.path}&amp;key=${header.key}"/>
<route>
```

Note: In the current version of the REST API, the prefix "/webhdfs/v1" is inserted as the value of
the path (specified before the query options in the URI of the webhdfs endpoint.)

Brief Background On The Aggregator Pattern
==========================================
The implementation of the Camel WEBHDFS component leverages many features of the Aggregator pattern. 
The aggregator is a generic component offered by Apache Camel. More information is available on the Camel 
Aggregator [here](http://camel.apache.org/aggregator2.html).

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
not make sense to write an unlimited number of messages to the same file because that file would grow too large. The 
completion condition will determine how files get broken up into reasonable size files that make sense 
for your environment. The values on these limits are configurable. You can control the frequency 
and size of files that the component writes into Hadoop by configuring values for completion conditions.  
The component will continue to append all messages belonging to the same message group, until either one of the 
following two completion conditions occur.

* aggregationTimeout – This configures the amount of time, in milliseconds, that a message group will remain active. The time measures idle time (no messages arrive.) Whenever a message arrives for a given message group, the timer starts over for that particular message group. If the timeout ever expires for a message group, the file will be written and no further messages will be appended to that file. 
* aggregationSize – The component keeps track of the total size of all messages sent to it for a given message group. Once this configured size is reached, the condition will trigger and the final message is written, and no further messages will be appended to that file.

As soon as one of the above limits is met for a given message group, the completion condition will fire. The message that caused the completion condition to trigger will be the last message written to the file. At that point, the file associated with the message group will be marked as done and no more messages are written to that file. Subsequent messages will be assigned to a new message group, and written to a new file.

Default Logic
=============
The logic used for path is as follows:
 * 'path' as-is if included in client request. Append 'path' to the end of 'default.data.dir' configured on the webhdfs endpoint.
 * If no 'path' header specified, set path value to be 'default.data.dir' configured on webhdfs endpoint.

The logic used for correlation key is as follows:
 * If no 'key' header specified, use 'path' header for correlation key.
 * If 'key' is included by client in request, append the 'path' property to this 'key' to arrive at final 'key' value.
 
More Examples
=============

The following route listens for incoming HTTP messages and routes to HDFS.

```xml
<route>
    <from uri="jetty:http://0.0.0.0:5160/ingestion/incoming"/>
    <to uri="webhdfs://localhost:50070/webhdfs/v1?path=${header.path}&amp;key=${header.key}&amp;aggregationSize=64000&amp;aggregationTimeout=3000"/>
<route>
```

The above route will allow client sending HTTP messages to specify query string that contains (optional) 'path' and 'key' 
parameters. This gives clients control over how messages are grouped into files and where files are written.

For example, if client sends a series of five messages sent through the route as follows. 

    > curl -i -X POST -T test.txt  "http://localhost:5160/ingestion/incoming"
    > curl -i -X POST -T test.txt  "http://localhost:5160/ingestion/incoming?key=A"
    > curl -i -X POST -T test.txt  "http://localhost:5160/ingestion/incoming?key=B"
    > curl -i -X POST -T test.txt  "http://localhost:5160/ingestion/incoming?key=C"
    > curl -i -X POST -T test.txt  "http://localhost:5160/ingestion/incoming?key=B"

The following lists the contents of directory where the files were written.
The default data location written to is '/user/fuse'. 
   
    > hadoop fs -ls /user/fuse
    > drwxr-xr-x   - fuse fuse  12 2013-09-10 09:45 /user/fuse/user_fuse_20131010142348492.txt
    > drwxr-xr-x   - fuse fuse  12 2013-09-10 09:45 /user/fuse/user_fuse_A_20131010142448495.txt
    > drwxr-xr-x   - fuse fuse  24 2013-09-10 09:45 /user/fuse/user_fuse_B_20131010142548499.txt
    > drwxr-xr-x   - fuse fuse  12 2013-09-10 09:45 /user/fuse/user_fuse_C_20131010142648534.txt

The first curl commands sends a message into the camel hdfs component without a key and
results in message getting its own message group (the component assigns a key based on default
path). Subsequent messages are sent through with key=A, B, then C. A final message is sent
again with key=B. Note the file created to hold messages with key=B is now twice as large as the others
indicating an additional messages was appended to this file. The component will continue
to keep these four message groups active until one of the completion conditions triggers.

A note on filenames: The filename that is generate is based on a combination of the path 
location, key, and a generated timestamp. The timestamp is based on when original message triggering
the new message group first arrived at component. The unique filename makes it easy to correlate 
messages to message groups.

Now we will send messages through with additional option for 'path' specifying where
the file is written. The specified 'path' location '/mytest1' is relative to the default data location '/user/fuse'.

    > curl -i -X POST -T test.txt  "http://localhost:5160/ingestion/incoming?key=X&path=/test1
    > curl -i -X POST -T test.txt  "http://localhost:5160/ingestion/incoming?key=Y&path=/test1
    > curl -i -X POST -T test.txt  "http://localhost:5160/ingestion/incoming?key=Z&path=/test1
    > curl -i -X POST -T test.txt  "http://localhost:5160/ingestion/incoming?key=Y&path=/test1

For message sent with only 'path' option specified, an internal 'key' is assigned under the covers
by the component (based on the path) and resulting in an additional message group and a new file gets created: 
    > curl -i -X POST -T test.txt  "http://localhost:5160/ingestion/incoming?path=/test1

Here is listing of the directory after sending the above five messages:

    > hadoop fs -ls /user/fuse/test1
    > drwxr-xr-x   - fuse fuse  12 2013-09-10 09:45 /user/fuse/test1/user_fuse_test1_X_20131010142448495.txt
    > drwxr-xr-x   - fuse fuse  24 2013-09-10 09:45 /user/fuse/test1/user_fuse_test1_Y_20131010142548499.txt
    > drwxr-xr-x   - fuse fuse  12 2013-09-10 09:45 /user/fuse/test1/user_fuse_test1_Z_20131010142648534.txt
    > drwxr-xr-x   - fuse fuse  12 2013-09-10 09:45 /user/fuse/test1/user_fuse_test1_20131010142348492.txt

With this flexibility of specifying key and path, the component can support
multiple client applications simultaneously. Each client can group its own messages to files in HDFS
and physically isolate that data according to its own requirements.
