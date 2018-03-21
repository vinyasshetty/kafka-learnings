## Consumer

* Maximum number of consumers you can have in a consumer groups should be equal to the number of partitions of a given topic.
* In other words, one partition cannot send data to more then one consumer within a consumer group.But we can have multiple partition sending data to one consumer within a consumer group.
* Data written to a topic can be read by multiple consumers groups,so make sure each our downstream app have thier own consumer group ,so that they can read the entirety of data from the topic.
* If we add a new consumer to a consumer group then this new consumer will get some partition allocated and the old consumer which used to get data from this partition will NOT get it anymore.Same happens when a consumer goes down in a consumer group or when admin adds a partition to a topic\(NOT a good thing ,due to chnages in partitioining of keys\).
* This process is called **"rebalancing"**,during this time whole consumer group becomes unavailable ,so we need to be careful  about these rebalancing.This will occur anytime when there is a change in the consumer count within a consumer group\(may be due existing consumers crashing or user adding new consumers\) or when new partitions or new topics gets added which are subscribed by running consumers.
* Consumers keep sending a regular heart beats to a consumer group co-ordinator broker.This may be different for different consumer group.It sends heart beat when it polls to get records and commits when it had read the message.
* If a consumer does NOT send a heartbeat long enough then the group co-ordinator will decide that consumer is dead and retrigger a rebalancing.We can control the frequency of heartbeats and duration after which group co-ordinator decides that consumer is dead.
* Consumer which wants to join a group sends a JoinGroup request to the GC\(Group Co-ordinator\) broker,the first consumer to join the group will be the leader and GC will send all ther consumers which info who want to join the group,then the leader consumer using a partitioing policy allocates partitions to the consumers and sends this info to the GC.Then GC sends only the required info a individual consumer saying from whicj partition they need to read from.ONly the GC and Consumer leader has the whole assignment list.This process will be done everytime when there is a rebalancing.&lt;Q**uestion** : So can the Consumer leader change everytime there is a rebalancing??&gt;
* You will need a KafkaConumer object with properties ,mandatory properties key.deserializer,value.deserializer and bootstrap.servers.good to always give group.id ,this will determine the consumer group name.
* A consumer can subsrcibe to multiple topics and we have a feature to use regular expression and whenever a new topic gets added ,then rebalancing will happen. The subscribe method has 4 overloaded types.
* Polling is the main method which controls co-ordinations heartbeats,rebalancing,fetching data,offset commit.
* poll method takes a Long which determines for how long\(in ms\) consumer will block before fetching data .This will return a ConsumerRecords\[K,V\] object which is a List/Iterable where each element corresponds to a ConsumerRecord from each partition it has read/fetched data for a particular topic.
* Always close the consumer ,this will make sure if the consumer dies ,GC comes to know about it asap.
* The poll loop does a lot more than just get data. The first time you call poll\(\) with a new consumer, it is responsible for finding the GroupCoordinator, joining the consumer group, and receiving a partition assignment. If a rebalance is triggered, it will be handled inside the poll loop as well. And of course the heartbeats that keep consumers alive are sent from within the poll loop. For this reason, we try to make sure that whatever processing we do between iterations is fast and efficient.
* When a consumer does a poll for the first time is when it joins a group.
* Rule is to have one thread or one application per consumer.Do NOT have multiple threads having consumer or multiple consumers in one thread. [https://www.confluent.io/blog/tutorial-getting-started-with-the-new-apache-kafka-0-9-consumer-client/](https://www.confluent.io/blog/tutorial-getting-started-with-the-new-apache-kafka-0-9-consumer-client/)

## Imp Properties

* Most of them are good enough to be keep in default.
* fetch.min.bytes =&gt; This will make the consumer to wait until it can fetch atleast this many bytes.This will reduce the two and forth movement of every small data.Look at cpu consumption of consumer and see if its very hgh when when very less data in topics then you can increase this value.
* fetch.max.wait.ms =&gt; default is 500 ms,max wait time before fetching.If you set fetch.max.wait.ms to 100 ms and fetch.min.bytes to 1 MB, Kafka will recieve a fetch request from the consumer and will respond with data either when it has 1 MB of data to return or after 100 ms, whichever happens first.
* fetch.max.bytes =&gt; Maximum data it can fetch per consumer.Below is per partition.One fetch can get data for multiple partitions,
* max.partition.fetch.bytes =&gt; Default is 1MB.This is the maximum amout of bytes a consumer can read per partition.Remember when you a do a poll,it a returns a Iterable ConsumerRecords which has ConsumerRecord object per partition read by that Consumer.No assume we have 20 partitions and we have 5 consumers,then each consumer can read maximum of 4MB,this may be very less and we may have a possibility of consumers dying and also we need to make sure this is greater then max.message.bytes which detemines max size of a message in broker.Also we need to be careful not to make this too high,beacuse then we will need a lot of time to process it and delaying the poll.
* Now consumer will keeps polling and also internally keeps sending heartbeats based on heartbeat.interval.ms\(default is 3 seconds\).Now if it does send a heartbeat or poll for a total of session.timeout.ms\(default is 10 seconds\) then GC will think this consumer is dead and it will rebalance.Also if a consumer is sending heartbeat but its not polling for a total of max.poll.interval.ms ,then also GC will consider consumer dead and rebalance.This is to avoid "livelock" situation
* max.poll.records =&gt; per poll how max many records it can take,&lt;Question max.partition.fetch.bytes and max.poll.records which one takes preference??&gt;
* auto.offset.reset=&gt;This property controls the behavior of the consumer when it starts reading a partition for which it doesn’t have a committed offset or if the committed offset it has is invalid \(usually because the consumer was down for so long that the record with that offset was already aged out of the broker\). The default is “latest,” which means that lacking a valid offset, the consumer will start reading from the newest records \(records that were written after the consumer started running\). The alternative is “earliest,” which means that lacking a valid offset, the consumer will read all the data in the partition, starting from the very beginning.
* enable.auto.commit=&gt;This parameter controls whether the consumer will commit offsets automatically, and defaults to true. Set it to false if you prefer to control when offsets are committed, which is necessary to minimize duplicates and avoid missing data. If you set enable.auto.commit to true, then you might also want to control how frequently offsets will be committed using auto.commit.interval.ms.
* partition.assignment.strategy decides which partition goes to which consumer.Default is RoundRobin.See below
* client.id can be any string,this is just to identify the consumer to the broker.&lt;Need to understand its usage&gt;??

## Partition Assignment to Consumer

* Assignment of a partition to a consumer is done with a class called PartitionAssignor.
* This class is extended by two classes : org.apache.kafka.clients.consumer.RangeAssignor and org.apache.kafka.clients.consumer.RoundRobinAssignor.
* In RangeAssignor, the topics and the consumers belonging to a group sent to the class and this one assigns a range of partitions to a consumer,say we have 5 partitions in a topic T1 and two consumers in consumer group g1 ,then first consumer will get 3 partitions and the last two partitions will go to the second consumer.
* In RoundRobinAssignor ,say we have 5 partitions in a topic T1 and two consumers in consumer group g1 ,then first consumer will get p1,p3 ,p5 partitions and p2 and p4 to second consumer .
* &lt;QUESTION&gt; : Read that assignment of consumer to partitions happens across topics in consecutive order in RoundRobinAssignor ,but assignment of partitions to a consumer is done by the consumer leader of that group,how does the consumer group know where the partition order stopped??

## Commit and Offset

* Kafka Consumers keeps track of the last message that they had read from partition.This process is called a "commit".
* We will have a \_\_consumer\_offsets topics which will have the offset ie the last read message from each partition by the consumers.\_
* Whenever a consumer rebalancing happens then the consumer wil go start reading the message based on the \_\__\_consumer\_offsets topic information,so here we can have potentially of duplicate message being read or some message being missed out to read if the offsets where not committed earlier._
* There are 4 types of offsets per consumer group per partition, a\)last commited offset\(This is is the offset committed in \_\_\_consumer\_\_offets topic, b\)Current Offsets : this is the offset from where the current reading is happening by consumer c\)High watermark Offset : This is the offset until which data has been replicated and is available for a consumer to read.d\)Log end offset : This is the total offset currently in the partition.

* Commiting of offsets to \_\__\_consumer\_offsets becomes important when rebalancing occurs ,because when a rebalancing happens ,all the consumers stops and gets thier partitions assigned ,now they will go and read from committed offsets to decide from where they need to start reading.So if the last committed offsets are done not correctly and is behind then it may cause duplication processing of data or if the last committed offset is ahead the consumer will miss reading some messages._

* Hence _ making sure the offsets are commited correctly and regularly becomes important._

* As_ eveything else important,commiting offset also happens as a part of "polling"  ie if you have set enable.auto.commit to true,then wherever a polling happens ,it sees if it has been **auto.commit.interval.ms** then a the **last poll\(NOT the current\) **offset is committed.Now with this we may have a problem when a poll method is called and it has not be yet auto.commit.interval.ms  ,then a processing will continue and then before the next poll if a rebalancing triggers then last poll processed records are not committed to offset.This will cause duplication._

* To avoid this we can set auto.enable.commit to false and we can commit offset progrmatically at the place and time we want.

* commitSync\(\) method helps is committing the** latest records that have been returned by poll method.**So make sure you call ** **this after all the processing is done in consumer.Also this method  will retry and this is blocking method call ,this will either commit and the move ahead else it will retry and if it still unable to commit the it will throw a error Error : CommitFailedException \(com.example.viny.Consumer1\)

* We can also do a asynchrnous commit ie a non-blocking commit.method is commitAsync\(\) . Now this is non blocking and also it will NOT retry .The reason it does not retry is that by the time commitAsync\(\) receives a response from the server, there may have been a later commit that was already successful. Imagine that we sent a request to commit offset 2000. There is a temporary communication problem, so the broker never gets the request and therefore never responds. Meanwhile, we processed another batch and successfully committed offset 3000. If commitAsync\(\) now retries the previously failed commit, it might succeed in committing offset 2000 after offset 3000 was already processed and committed. In the case of a rebalance, this will cause more duplicates.

* Once the commitAsync\(\) method is done ie either it is succesful or it can fail then it will call a callback method.\(com.example.viny.Consumer2\)

* We can combine sync and async methods in a consumer.We can have a commitSync at the finally part ie when the consumer is closing due to failure or rebalancing and use commitAsync otherwise.**&lt;Question : When a rebalancing happens then a existing consumer does NOT go outside of the existing "loop",so combining like we did in \(com.example.viny.Consumer3\) is still suspectible to duplication???&gt;.**

* Now these above commit methods will commit directly the whole ConsumerRecords that have been read by the "poll",but we can further control this and we can explicitly commit offsets by sending a Map\[TopicPartition,OffsetAndMetadata =&gt; \(com.example.viny.Consumer4\) .

* If a consumer is seeking for a offset which does NOt exist,then it will behave based on "auto.offset.reset" ,it does NOT throw a exception but if you try to get data from a topic which has does NOT have a particular partition then we get a exception ** java.lang.IllegalStateException: No current assignment for partition cards-12 **

**&lt;HAVE TO COME BACK AND SEE HOW ONE CONSUMER READING FROM TWO TOPICS BEHAVE ON REBALANCING&gt;**

## Rebalance Listeners

To answer my above question as to how to do some activities before the rebalancing happens is answered by "Rebalance Listeners"

So we need to have a class which extends ConsumerRebalanceListener and extends below two methods:

```
/*Below will be run it knows a reblanacing should happens but before it actually happes*/
public void onPartitionsRevoked(Collections<TopicPartition> partitions) 
//Below will happen after the rebalancing has happened but before the new consumer starts reading.
public void onPartitionsAssigned(Collection<TopicPartition> partitions)
```

Now the object that we create from class which extends ConsumerRebalanceListener needs to be passed in the subscribe method.=&gt; \(com.example.viny.Consumer5\).Even when u restart a Consumer ,it will go into above methods.

We have a consumer.seek\(t:TopicPartition,offset:Long\) ,this will make the consumer to read from this given offset.

We have seek method on consumer which will help us to start reading from the point you have given in the seek when the next poll comes. seek does NOT alter the last committed directly.

consumer.seek\(t:TopicPartition, offset: Long\)

Usually added at the "onPartitionAssigned" method in ConsumerRebalanceListener.

**&lt;Will Come back to learn more about shutdown hook to understand about WakeupException and write a complete Consumer Code&gt;**

