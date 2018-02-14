# **Topics**

* Topics is a append only commit-log.
* Messages/Events that are sent to a topic can only be appended.ie data once written to a topic is immutable.
* Data written to a topic should be of same schema

`bin/kafka-topics.sh --zookeeper localhost:2181 --describe --topic test1`

`Topic:test1    PartitionCount:2    ReplicationFactor:2    Configs:`

`Topic: test1    Partition: 0    Leader: 0    Replicas: 0,1    Isr: 0,1`

`Topic: test1    Partition: 1    Leader: 0    Replicas: 1,0    Isr: 0,1`

# **Broker **

* Single Kafka Server is called a Broker.
* Broker receives messages from producers,then assigns them a offset and commits the message to storage on disk.
* It also services consumers who want to fetch data from a given topic and from a given partition and a given sequence.
* Broker.id is the unique name given to each broker in config/server.properties.
* If Consumers and Producers wants to read/write messages then they always talks to the Broker which is the leader of the partitions.Replicas of the partitions are just used when the Broker which is the leader for a given partition fails.
* In config/server.properties we have the log.dir which determines where the data that you write into kafka will be saved.

* One Broker among the cluster is automatically selected and that broker acts as a administrative broker\(controller\) which decides which assigning partition to a broker ,which broker will be the leader for a given partition ~~and also monitors brokers for failures.

## 

# Consumer

To Create a Consumer via API:

First have java.util.Properties and set below properties.

```
p.setProperty("bootstrap.servers","127.0.0.1:9092")

p.setProperty("key.deserializer","org.apache.kafka.common.serialization.StringDeserializer")
p.setProperty("value.deserializer","org.apache.kafka.common.serialization.StringDeserializer")
p.setProperty("group.id","test1")
p.setProperty("enable.auto.commit","true")
p.setProperty("auto.commit.interval.ms","1000")
p.setProperty("auto.offset.reset","earliest")

val consumer = new KafkaConsumer[String,String](p)
consumer.subscribe(List("third-topic"))

while(true){
  val records = consumer.poll(100)
  for(record <- records){
    println("Key is " + record.key())
    println("Value is " + record.value())
    println("Partition is " + record.partition())
    println("Offset is " + record.offset())
  }
}
```

## Configurations

Kafka uses Key-Value pair for configuration in properties File.The essential configurations are the following\(See BELOW\):

* `broker.id`
* `log.dirs`
* `zookeeper.connect`

**We have server\(Broker\) level configs that can be set in server.properties like the compression to be used,retention policy of the data etc but these can be overwriiten either while creating a topic,producer or consumer based on its availability.**

# **Broker Configs**

* **broker.id** should be a integer value and it should always be unique for every broker within a cluster.Deafult to 0
* **port **this is default set to TCP port 9092.
* **zookeeper.connect** : semicolon  separated list of `hostname:port/path` strings.if we dont give path then "/" root path is selected in zookeeper znode.Its always good a give list of all the nodes in zookeeper ensemble.
* **logs.dirs **: This is a comma separated list of directory locations .Directory is chosen based on "least used"\(NOT in terms of bytes size but the number of directories\)
* If we have log.dirs=/tmp/kafka-logs-1  .So once u create a topic then we will have a directory called **/tmp/kafka-logs-1/&lt;topicname&gt;-&lt;partition\_count&gt;/** will be created.  
* **num.recovery.threads.per.data.dir **=&gt; This determines the thread that will be used per directory mentioned in logs.dir. Default is 1.If we have set this to 10 and if we have 3 directories mentioned in logs.dir then number of threads used is 30.Main use of this thread is to open a log segment file or to cleanly close the log segment file or when starting after a failure then to check and truncate the files.\(truncating of large partitions might take a lot of time if this is set to default of 1\)
* **auto.create.topics.enable **=&gt; If set to true\(default\) then This will create a non existing topic when a consumer or producer tries to read or write from this topic or when client asks for metadata information from this topic.
* **num.partitions** will be used when a broker is created due to above point.**Now how do we select the number of partitions **:Usually we can have atleast the partition count to be same as number of brokers.But it can depend on other fetaures also ,say we need a write/read speed from a topic to be 2 GB/sec ,but one consumer can read at the speed of say 100 MB/sec ,the \(2GB/sec\)/\(100MB/sec\) =&gt; 20 .So we have 20 partitions then we can at max have 20 consumers with each reading 100 MB/sec and thus giving us a speed to 2GB/sec.

These below are given for broker wise configs,but these can be changed individually for TOPICS also.Check Docs.

* **log.retention.ms/log.retention.hours/log.retention.minutes **=&gt; If all are set then ms since its the smallest unit takes preference. ~~** Is this the time of each message that is calculated or the last change timestamp of the log file\(which may contain several message\) which is taken into account?**~~**See the answer to this after two points below.**
* **log.retention.bytes** =&gt; Now this determines how big each partition can get before deleting the partition.Say this is set to 2000000000 \(2 GB\) ,and that topic has 10 partitions,then that topic can have at the max 20GB of data.But the tracking and deleting is done in terms of partition ie if a partition size exceeds greater then 2gb then its cleaned up.
* If we have set bot time and bytes retention then which ever condition is being satisfied first that clean up takes place.
* **log.segement.bytes** determines the size of each log file for a given partition,by default this is 1gb,so until this file size is 1gb messages going to this partition can be written to this file.Once it reaches 1gb,then the file closes and the log.retention.ms time clock starts.We need to be careful here since if the file size is not reaching 1gb\(log.segment.bytes\) then  the file handle is kept active and log.retention.ms countdown is NOT started since file handle is active.
* Giving a smaller log.segment.bytes might be problematic because the file handle needs to frequently opened and closed whenever file size reaches log.segment.bytes.Point to **Note here is file handler closes only once file size reaches log.segment.bytes and only when file handler closes is when log.retention.ms countdown starts.**
* **log.segment.ms **=&gt; This will determine each file handle should be active for how long after which handle is closed and messages are written to a new file.Again as above if log.segment.ms and log.segment.bytes are both set then whichever condition reaches first that closes the filehandle. 
* **message.max.bytes** =&gt; This determines the maximum size of each message.If compression is used then the compressed file should be smaller or equal to max.message.bytes else the broker will throw a error.Keeping this large make it suspectible to failure since the thread that is writing data needs to active for longer.~~Also use this in co-ordination with producer client fetch.message.max.bytes because if fetch.message.max.bytes  is smaller then message.max.bytes ,then the consumer which encouters larger messages gets stuck.~~&lt;EDIT&gt; Seems Deprecated.

## Topic Configs

`bin/kafka-topics.sh --zookeeper localhost:2181 --create --topic test2 --config compression.type=gzip --config cleanup-policy=delete --config retention.ms=10000`

**This will show only the overwritten values and not default value from server properties**

```
bin/kafka-configs.sh --zookeeper localhost:2181 --entity-type topics --entity-name test2 --describe
```

retention.ms and file.delete.delay.&lt;EDIT&gt;

**Values can be overwritten or altered:**

`bin /kafka-configs.sh --zookeeper localhost:2181 --entity-type topics --entity-name test2 --alter --add-config  retention.ms=6000`

**To delete a parameter :**

`bin /kafka-configs.sh --zookeeper localhost:2181 --entity-type topics --entity-name test2 --alter --delete -config  retention.ms`

## Producer Configs

batch.size , linger.ms  ,buffer.memory,max.block.ms. &lt;EDIT&gt; Need to understand dependency.

acks

&lt;EDIT&gt;

## Consumer Configs

* auto.offset.reset =&gt; earliest ,latest\(default\),none .If earliest is set then if we consumer belongs to a group id ie\(group.id is set\) even then it strats from beginning.none =&gt; will throw a error basically is group.id is NOT set.
* group.id =&gt;   This will make the consumer part of a consumer group\(ie Name that you give\). This will basically create a offset topic which will keep track of your latest read partition offset.This feature loses relevance if auto.offset.rest is earliest.
* fetch.min.bytes,fetch.max.bytes

 

## General

* Retention of the data on a broker can either be determined by a Broker properties or we can have specific topic related retention policy set.
* Retention of the data can be set based on Time of the data or also on size of the topic.

## Zookeeper

* Zookeeper holds metadata information about the broker,topics,partitions.
* After you start zookeeper ,say nc localhost 2181 ,this will connect to zookeeper then say **srvr ,this is a 4 letter command for zoopkeeper which will give you details about the zookeeper.**
* Zookeeper cluster is called a **ensemble. **We usually select odd numbered ensemble like 3-node ensemble.This is called quorum.If we have 5 node ensemble then until 3 nodes are active zookeeper can still work ie Zookeeper ensemble works fine until maximum \# of nodes are active in that ensemble.
* To build a zookeeper ensemble,we need to update below in zookeeper conf files ie :
* All the conf files needs to have details about other zookeeper servers and also the myid file in dataDIr. &lt;EDIT&gt;
* Zookeeper with kafka : It helps in selecting the controller broker, it has all the metadata info about the topics, information  about the broker health.



