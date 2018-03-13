* Kafka uses zookeeper to maintain the list of brokers.
* Every broker has a unique id called broker.id. Whenever a broker starts it registers itself with a ephermal node in zookeeper.

  So in broker server.properties we will have something like :

```
broker.id=1
zookeeper.connect=zoo1hostname:2181/kafka/,zoo2hostname:2181/kafka

//Zookeeper provides a lightweight distributed hierarchial filessytem. 
//Now in kafka znodes we will end up having /kafka/brokers/ids. By ephermal we mean if broker 
//goes down for some reason then /kafka/brokers/ids this cleaned up in zookeeper.This is how 
//zookeeper keeps track of which brokers are alive
```

```
cd /Users/vinyasshetty/Downloads/kafka_2.11-1.0.0/bin
//Below will list the directories in root of zookeeper
Vinyass-MacBook-Pro:bin vinyasshetty$ sh zookeeper-shell.sh 127.0.0.1:2181 ls /  
[cluster, controller_epoch, controller, brokers, zookeeper, admin, isr_change_notification, consumers, log_dir_event_notification, latest_producer_id_block, config]

 Vinyass-MacBook-Pro:bin vinyasshetty$ sh zookeeper-shell.sh 127.0.0.1:2181 ls /brokers
 ** [ids, topics, seqid] **

 Vinyass-MacBook-Pro:bin vinyasshetty$ sh zookeeper-shell.sh 127.0.0.1:2181 ls /brokers/ids 
 [0,1]      /*Since i am running two brokers and This ephermal ie if connection of broker to zoopkeepr
              goes away then the corresponding value(either 0 or 1) gets deleted*/


 Vinyass-MacBook-Pro:bin vinyasshetty$ sh zookeeper-shell.sh  127.0.0.1:2181 get /brokers/topics/cards/partitions/1/state
Connecting to 127.0.0.1:2181

WATCHER::

WatchedEvent state:SyncConnected type:None path:null
{"controller_epoch":14,"leader":1,"version":1,"leader_epoch":5,"isr":[1,0]}
cZxid = 0x7d4
ctime = Fri Mar 09 14:23:49 CST 2018
mZxid = 0x935
mtime = Sun Mar 11 15:07:55 CDT 2018
pZxid = 0x7d4
cversion = 0
dataVersion = 10
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 75
numChildren = 0
Vinyass-MacBook-Pro:bin vinyasshetty$ pwd
/Users/vinyasshetty/Downloads/kafka_2.11-1.0.0/bin
Vinyass-MacBook-Pro:bin vinyasshetty$ sh zookeeper-shell.sh 127.0.0.1:2181 get /brokers/ids/0
Connecting to 127.0.0.1:2181

WATCHER::

WatchedEvent state:SyncConnected type:None path:null
{"listener_security_protocol_map":{"PLAINTEXT":"PLAINTEXT"},"endpoints":["PLAINTEXT://192.168.1.125:9092"],"jmx_port":-1,"host":"192.168.1.125","timestamp":"1520798874231","port":9092,"version":4}
cZxid = 0x91a
ctime = Sun Mar 11 15:07:54 CDT 2018
mZxid = 0x91a
mtime = Sun Mar 11 15:07:54 CDT 2018
pZxid = 0x91a
cversion = 0
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x16216ad00c30001
dataLength = 196
numChildren = 0
Vinyass-MacBook-Pro:bin vinyasshetty$
```

Also if you try to register a new broker with a same existing broker id,zookeeper will complain.

## Controller

* This is one of the broker in the cluster.At a time we can have only one broker.
* The broker which joins the cluster first becomes the controller and it creates a controller ephermal node.
* Now other broker when they try to create a controller znode they will get a exception saying controller already exist.Now these brokers create a watch on this znode to let them know if it goes down anytime.
* Now the controller broker keeps a watch on the "brokers/" znode information and whenever a broker goes down ,then it gets the list of all the partitions for which the broker was a leade**r\(how does it have this info??\) a**nd then based on where the repliacs are it will select a new broker and send that information the new leaders of the partitions and also its followers about this chnage.
* Now the new leader brokers know that they need to server the consumer and producers for that partition and the follower broker know they should replicate the messages.
* If a new broker is joining the cluster ,then the controller uses the new broker's broker id to see if there are replicas of any partitions and if it has then it notofies other broker who the are leader and followers of the partitions ,then this new broker will start replicating the partition from them.
* Controller knows for every partition which is the leader broker and which brokers have sync replicas for that partition.

```
Vinyass-MacBook-Pro:bin vinyasshetty$ sh zookeeper-shell.sh 127.0.0.1:2181 get /controller
Connecting to 127.0.0.1:2181

WATCHER::

WatchedEvent state:SyncConnected type:None path:null
{"version":1,"brokerid":1,"timestamp":"1520798869366"}
```

## Replication

* Every partition is replicated in kafka.Rule is Kafka gives a gurantee that your data in a partition will be there until N-1 brokers fails.
* So that means says i have 3 brokers ,then my partitions will be there until 2 brokers goes down.To provide this gurantee,the number of replication factor per partition that you can provide can be only upto to \(&lt;=\) Number of brokers.You cannot have two brokers and 3 replication factor for a partition/topic.
* Every replica is classified into **leader** and** follower .**
* **leader keeps track of which all followers are in sync and out ofsync**.Way it does this is each follower broker sends Fetch request to the leader same like how consumer does.it always asks the request in order ie if msg 1 and 2 is replicated only then it will ask for msg 3.This way the leader knows where the follower is based on its request.
* Leader makes the follower to be out of sync if it does NOt send a fetch request in 10 seconds or if it sends a fetch request but its not update till the last offset .
* **followers which are out of sync will NOT be eligible to become leader if the leader broker goes down. **time after which a follower is considered out of sync as above is based on value **replica.lag.time.max.ms**

## Requests

* We have 3 main types of requests : Produce ,Fetch and Metadata.
* Each broker has a metadata cache which has information about each topics ,its correspoding partitions along  its leaders and followers .** &lt;Question : How does eah broker have this updated metadata cache?&gt;**
* A client will connect to any broker that you have given in your code\(bootstrap.servers\) and it will send a Metadata Request,now the client will know which is the Leader for a given partition and it will send a produce or fetch request based on that.Next it will use this info to send requests.
* Now the client will keep the metadata information upto date by resending the request for every **metadata.max.age.ms** or when a client sends fetch/poduce request but gets a exception saying "Not a leader" from the broker.
* During Producer request ,if acks=-1/all ,then once the leader gets the data ,it will store the ack request in a queue called "purgatory" untill all the followers are upto date.
* Fetch : this is sent by consumer clients or by follower replias brokers.It will usually be a requests aksing data from multiple topics partitions offsets, like from topic t1 ,partition 0 ,offset 10, from topic t2 partition 1 offset 12 \(with assumption that this broker is the leader for these partitions\).
* Clients can specify the min and max amount of data it can recieve\(look at consumer chapter fo configs to control this\)
* **Can Producer also control amount of information it can send?\[**\[Answer : Well using linger.ms we can make the producer to fill its batch.size before sending the messages\]I know broker has max.message.bytes to say how much max information per message can it be.
* If a client sends a fetch request for a topic or partition which does not exists,it will get a error.
* **Consumers and Producers can only fetch and produce requests to leader brokers.**
* **Consumers can only fetch data which has been replicated completly,this may cause consumers to wait until sync happens,but the delay can be limited by replica.lag.time.max.ms**

One Partition cannot be split be split between multiple broker.It needs to fit in one broker.

```
Say in log.dirs=/dev1/data1,/dev1/data2
```

* Selection of a brokers when creating partitions for a topic .
* Say we have 6 brokers and we want a topic with 6 partitions and replication factor of 3.Now total number of partitions would be 18.
* **How is the leader of each partition selected?**??? **Is it Random??**.Lets randomly take broker 3 to be Leader partition 0, broker 4 to have Leader partition 1,broker 5 to have Leader partition 2 ,broker 0 to have Leader partition 3 and so on.
* Now for replicas,partition 0 has leader in broker 3,so partition 0 replicas will be broker 4 and 5.Similarly for partition 1, broker 4 was the leader so replicas will be broker 5 and broker 0.
* Was kafka is rack aware also,in that case say we have broker 0,1,2 in rack 0 and vroker 3,4,5 in rack 1 .then in terms of leader it will follow same process.But for replica . Seq order will 0,3,1,4,2,5.
* Now if logs.dir has multiple directories,it choses the directories which least number of sub-directories.
* Within a topic-partition directory kafka lets us write messages we send into a file which it calls a segment.Now this segment file can grow upto a maximum size\(log.segment.bytes\) or upto certain time\(log.segment.hours etc\).ONce either one of them reaches,it closes that segment and creates a new one.
* **Once this segment closes ,then these files are eligible for clean up based on log.retention.bytes or log.retention.ms/etc.Important point to note here is these retention policies starts to apply once the file handle is closed,in other words "active segment" can never be deleted.**
* We can mention the compression type if we want while writing data and while fetching no need to metion.

* A message when its written ,below are some of the info:

```
Offset - 8 bytes
msg length - 4 bytes
CRC - 4 bytes
magic byte - 1 byte
attribute  - 1 byte
timestamp - 8 bytes
key length - 4 bytes
key content - varies
value length - 4 bytes
value content - varies
```

Now we can view the contents of the segment file using command :

```
sh kafka-run-class.sh kafka.tools.DumpLogSegments --print-data-log --files /tmp/kafka-logs-1/cards-0/00000000000000000000.log  --deep-iteration
```

Kafka combines messages based on batch.size and linger.ms and send the batch together to broker.So make sure max.message.bytes is greater the batch.size.

If a consumer wants to seek a message ie by giving a TopicPartition and a offset,then kafka needs to figure out from possible multiple segment file as to where the offset is,so for that reason kafka maintains a index file per segment which helps in easy determiming of a offset .

```
cards is the topic name ,0 is the partition and logs.dir=/tmp/kafka-logs-1 in broker server.properties
Vinyass-MacBook-Pro:bin vinyasshetty$ ls -l /tmp/kafka-logs-1/cards-0
total 16
-rw-r--r--  1 vinyasshetty  wheel  10485760 Mar  9 14:23 00000000000000000000.index
-rw-r--r--  1 vinyasshetty  wheel       149 Mar  9 14:26 00000000000000000000.log
-rw-r--r--  1 vinyasshetty  wheel  10485756 Mar  9 14:23 00000000000000000000.timeindex
-rw-r--r--  1 vinyasshetty  wheel         8 Mar  9 14:25 leader-epoch-checkpoint
Vinyass-MacBook-Pro:bin vinyasshetty$
```

These indexes can be regenrated by itself using the segment log file.

```
sh zookeeper-shell.sh 127.0.0.1:2181 get /brokers/topics/cards_1/partitions/1/state
Connecting to 127.0.0.1:2181

WATCHER::

WatchedEvent state:SyncConnected type:None path:null
{"controller_epoch":14,"leader":0,"version":1,"leader_epoch":0,"isr":[0,1]}
/* So all the information related to a partition is available
```

## Compaction

* Log Segment whose file handle has been closed ie no more messages are written to them are eligible for clean up.
* Now kafka provides two types of clean up support "compact" or "delete" ,this can be selected by setting log.cleanup.policy .default is "delete"
* now "delete" is straight forward as we discussed and it can be set with time or space as log.retention.ms/hours/minutes etc or log.segment.bytes and which is satisfied first then deletes that segment file altogether.
* Now in "compact" mode, the segments in a partition are classified into a "head" and "tail".Any segment whose file handle is open and if file handle is closed and if messages are not older then **log.cleaner.min.compaction.lag.ms** then those will be treated as the head part ,rest of the segment in the partition will be treated as "tail". Now kafka has compaction manager and bunch of compaction threads\(log.cleaner.threads\) which will read msgs from "head" part of the segment and create a map which has the 16 segment bytes having the key from the message and 8 segment value part which will store the offset of the latest key .Each thread will have its own map and the total size for the map across all threads allowed is log.cleaner.dedupe.buffer.size bytes.
* Once this map is ready,we will now read from the tail,if the key from the tail is NOT available in the map ,then thats part from tail is copied over into a segment called "replacement segment" ,if the key from a message in the tail is available in the map keys then that is not copied over,once the whole tail is read then tail segment is deleted and replacement segment becomes the new segment.Point to note is when messages from tail is copied to replacement segment,its offsets are NOT changed,also the head part of the segment is NOT changed.
* Now the segments will have only one message per key in that partition.
* Kafka also supports one extra feature in compaction ie say if you send a msg with existing key with its value as null,the once compaction is done ,we will have the message with that key and value as null in the segment.This is called a **tombstone** or a **delete marker.Even if we send any future message with the same key and some value,after compaction the message where the key is null is retained.Now after the duration delete.retention.ms then the msg is deleted altogether.So any consumer planning to read from the beginning has to make sure he reads it before delete.retention.ms.**
* So when does compaction run and how does it choose on which partition in the topic it has run on? Well compaction runs when the ratio of log head to log tail is greater or equal to **min.cleanable.dirty.ratio ** and whichever partition has higher ration it will run on them first.
* ** Compaction works on topics that key and values if null key\(ie you dont pass any key \) then it will throw a error .**



