## Producer

To Create a Producer via API:

First have java.util.Properties and set below properties.

`p.setProperty("bootstrap.server","127.0.0.1:9092")`

`p.setProperty("key.serializer","org.apache.kafka.common.serialization.StringSerializer")`

`p.setProperty("value.serializer","org.apache.kafka.common.serialization.StringSeriailzer")`

`p.setProperty("acks","1")`

Possible values \[all, -1, 0, 1\]. This is a String Type. all/-1 means Producer will wait for ack until all the replication is done successfully.0 Producer does not wait for ack and 1 means Producer waits for ack only until data is reached successfully to leader broker partition.See The docs for how this impacts on retries variable.

`p.setProperty("compression.type","gzip") // Deafult is none,compression gzip,snappy,lz4 or none.`

`val p1 = new KafkaProducer[K,V](p) // org.apache.kafka.clients.producer.{KafkaProducer,ProducerRecord}`

`val r1 = new ProducerRecord[K,V](<topic name>,<key>,<value>) /* K and V are types of key and value*/`

`p1.send(r1)`

`p1.close()`

close is important because Producer batches records in the buffer before sending itto broker,close makes sure records from buffer are flushed out before the progams dies.

 ** IMPORTANT : As seen above ProducerRecord determines which topic to write data to,so we can create one KafkaProducer and then multiple ProducerRecord can write to different topics,provided key value types are same. **

* **ProducerRecord **is the basic one we need.This always needs a topic to which we want to send data to data\(value\) and also the data\(value\).Optionally it can also take key and/or partition\(Int\) and/or timestamp\(Long\).
* FIrst thing that happens is serialization of keys and values to ByteArrays so that they can be sent over the newtork.
* Next the key and value is sent to Partitioner but if the partition value is already given then it returns the same value else it will return a value based on the key.
* Now the Producer knows to which topic and to which partition it needs to send data to.This data now is a added to batch\(created for every partition in a topic\),then a different thread send this batch to the Leader Broker of that corresposnding partition.If messages are written to Broker successfully then Broker returns a **RecordMetadata Future **object\(based on acks value\) ,with the topic,partition and the offset of the record\(&lt;EDIT&gt; since we are sending in batches,all offset sent?\) ,if it fails then a Error is sent and it can retry to resend it.\(based on retires value ,default to 0\)

**To Create a Producer via API:**

First have java.util.Properties and set below properties.These 3 are mandatory

`p.setProperty("bootstrap.server","127.0.0.1:9092,127,0.0.1:9093") //Atleast include two so that if one broker goes down then Producer can still connect`

`p.setProperty("key.serializer","org.apache.kafka.common.serialization.StringSerializer")`

`p.setProperty("value.serializer","org.apache.kafka.common.serialization.StringSeriailzer")`

* Kafka expects data of the key and the Value to be ByteArrays but the Producer API can take regular Java Types and Producer Internally converts them into ByteArrays.  key.serializer and value.serializer should be set a class which would extend from org.apache.kafka.common.serialization.Serialization Interface. Partition and Timestamp does NOT need a serializer because it already has a fixed type of int and long respectively.**Even if we dont send a key ,we still need to set the key.serializer.**
* Even before sending a message Producer can get different exception like:

  * SerializationError 
  * BufferExhaustedException
  * TimeOutException

* Producer send method takes a ProducerRecord object.

* send method returns a Future object on which we can call a get,get on a Future is a blocking method.[\(Read\)](http://www.baeldung.com/java-future).This is sync/blocking way of sending.

* The send method returns a **Future\[or.apache.kafka.clients.producers.RecordMetaData\] **,usually we do NOT need to access this RecordMetaData but we do want to access the Future for Exceptions ,so we have a method to register a callback on this Future via send method itself.

* If the message is written successfully then the get method on Future returns a RecordMetadata Object.This object has method to get the offset,partition,topic etc.

* Kafka Producer has two types of error :One that it can retry to send\(Like connection error\) and the other where it cannot/will not retry like size of the message being very large.

* If we set to acks=0 and then try to get the Future\[RecordMetadata\],we will get partition info but the offset information will be set to -1,because as soon as the producer sends the message ,it will get back RecordMetadata saying message has been successful due to acks being 0,and in the Producer we will have the partition info coming from the partitioner but then NOT the offset since the message may or may not have reached the broker.

* With acks = 1,there is still two possibilties of message being lost ie 1\)Msg written to leader but the leader broker goes down and the the replices broker will be made leader if unclean leader election is true. 2\)Now same scenario as 1 but if the unclean leader is false even then replicas might not be marked as async yet since duration is within 10secs\(**replica.lag.time.max.ms\).**

* With acks=1 , if we send msgs in sync then there will delay for roundabout time,but we if send in async ,delay will be less but still there will be some delay based on how many msgs we can send before we start getting those msgs ack.

* buffer.memory =&gt; Amount of memory which producer will use to buffer the messages before sending to broker.This is for the whole java producer client.If the buffer gets full then send method will be blocked until the duration of **max.block.ms **and after that a exception will be thrown.

* compression.type =&gt;Deafult is none,compression gzip,snappy,lz4 or none.Data is compressed **before** sending to brokers.

* Snappy has better cpu performance but less compression ratios compared to gzip.

* retries =&gt; This will determine how many times the producer will retry to send the messages when it recieves a ERROR.This **Error has to be retriable error ie If we get a error for serializationexption or messages too long such ones are not retried**.Retrying by default is done after 100ms but that can be controlled with `retry.backoff.ms.`Mostly we see errors happening due to broker being down temporarily,so be be careful while setting retry.backoff.ms and make sure its atleast greater then time required to broker to come up or a new Leader selelction.This will NOT matter if acks=0 . &lt;EDIT&gt; **if the retries are set to say 3,then if the message sending is failed for the first time ,but ends up succedding at the 2nd attempt will the first error be printed in the first callback\(Assuming  we are printing it \) of the send method?or the callback of the send method will be called only after the retries?\[See the ANSWER below for this\]**

* Retries are handled internally in the producer before the callback is invoked. The callback should only ever be invoked once the producer is completely done with the record, either because it was acknowledged or because it gave up \(timed out and no further retries\). So by the time you are printing the stack trace, it has already retried as many times as you requested and the last exception that ultimately caused it to give up was an UnknownTopicOrPartitionException.The producer will retry sending as long as it has retries left and the exception is a RetriableException. UnknownTopicOrPartitionException is a RetriableException, so it should be retried as many times as the \`retries\` setting you use for the producer.

* batch.size =&gt; KafkaProducer will batch messages to send to same partition of a topic and then send the batch.This parameter determines the **size of the batch in bytes for a given partition of a topic**.The way kafka does batching is as soon as it receives a message it will try to send it if send method is NOT busy but if the send method is busy it will start collecting the messages belonging to a  given partition into batches until batch.size .Point to NOTE is Producer does NOT wait for the batch to be full,if the send method thread is free then even one message batch is sent.We also have a parameter called **linger.ms **,now if the batch is NOT full and if the send method is ready to send the messages then it will wait for linger.ms duration to collect some more messages before sending the batch.**Here by send method i mean the thread which sends the msg from Producer to Broker and the actual producer send method.Also my understanding is messages in the batch are committed as a "transaction" ie either full batch is commited or nothing from the batch s committed.**

* client.id =&gt; This is a string which is used to identify the client which is sending messages to the broker.**&lt;EDIT&gt;**Have to see how this is useful,will the messages that is sent to broker have this information retained anywhere?

* buffer.memory,batch.size,acks,retries,retry.backoff.ms,callback,get,linger.ms,max.block.ms,max.in.flight.requests.per.connection,max.request.size

* max.in.flight.requests.per.connection =&gt; Maximum number of messages it Producer will wait until it recieves a acknowledgement.Say this is set to 5 and if acks is non-zero then ,Producer can send upto 5 messages and then it will start waiting for ack. If this set to 1 then we can have ordering being maintained even if we have exceptions and have a non-zero retries.**&lt;EDIT&gt;**Does this end up acting like sync sends with retries assuming sync send always retries for a failure before moving onto next message?

* max.block.ms :=&gt; Max amount of ortime producer will block send method when buffer is full or when partitionsFor\(\) method is not returning due to metadata not being avaiilable

* request.timeout.ms =&gt; The configuration controls the maximum amount of time the client will wait for the response of a request. If the response is not received before the timeout elapses the client will resend the request if necessary or fail the request if retries are exhausted.Default is 30 secs**.Does this time start from the point msgs are placed on buffer or after the point msgs have been sent out from the buffer?**

* max.request.size =&gt; This basically means the maximum size of request in bytes the Producer can send to broker per request.ie if max.request.size is set to 1MB and max.message.bytes on broker is also set to 1MB then,producer can send a message maximum size of 1 MB or it can batch 1000 messages of 1K size each.Broker also has value which tells broker whats maximum size of the message it can receive per connection ie message.max.bytes.  So make sure these are in sync.**&lt;EDIT&gt; **Wonder what's the difference between max.request.size and batch.size.May be batch.size is more logical that it will batch  bytes if possible before sending but max.request.size is more fixed and it will throw a exception if we have a larger message/batch ie say linger.ms is kept to 10000\(ie 10 sec\) and say batch.size is 20MB  and say max.request.size and max.message.bytes is 10MB.Now if our client collects 20MB worth of data for a given partition in 10 seconds then this will fail with a Exception ????

* [https://stackoverflow.com/questions/43601747/kafkaproducer-difference-between-callback-and-returned-future/43667728](https://stackoverflow.com/questions/43601747/kafkaproducer-difference-between-callback-and-returned-future/43667728)

* If we have retries to non zero and max.in.flight.requests.per.connection to greater then 1 and sending in async mode then t**here is chance of ordering being lost for a given partition** ,to rectify that we need set max.in.flights.requests.per.connection to 1

* **Increase Throughput of Producers **: Use compression,increase batch.size ,increase linger.ms and also increase buffer.memory.

Serialization : Look at the Schema Registry Part.

