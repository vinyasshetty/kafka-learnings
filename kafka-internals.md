* Kafka uses zookeeper to maintain the list of brokers.
* Every broker has a unique id called broker.id. Whenever a broker starts it registers itself with a ephermal node in zookeeper.

  So in broker server.properties we will have something like :

```
broker.id=1
zookeeper.connect=zoo1hostname:2181/kafka/,zoo2hostname:2181/kafka

//Zookeeper provides a lightweight distributed hierarchial filessytem. 
//Now in kafka znodes we will end up having /kafka/brokers/1. By ephermal we mean if broker 
//goes down for some reason then /kafka/brokers/1 this cleaned up in zookeeper.This is how 
//zookeeper keeps track of which brokers are alive
```

Also if you try to register a new broker with a same existing broker id,zookeeper will complain.

## Controller

* This is one of the broker in the cluster.At a time we can have only one broker.
* The broker which joins the cluster first becomes the controller and it creates a controller ephermal node.
* Now other broker when they try to create a controller znode they will get a exception saying controller already exist.Now these brokers create a watch on this znode to let them know if it goes down anytime.
* Now the controller broker keeps a watch on the "brokers/" znode information and whenever a broker goes down ,then it gets the list of all the partitions for which the broker was a leade**r\(how does it have this info??\) a**nd then based on where the repliacs are it will select a new broker and send that information the new leaders of the partitions and also its followers about this chnage.
* Now the new leader brokers know that they need to server the consumer and producers for that partition and the follower broker know they should replicate the messages.
* If a new broker is joining the cluster ,then the controller uses the new broker's broker id to see if there are replicas of any partitions and if it has then it notofies other broker who the are leader and followers of the partitions ,then this new broker will start replicating the partition from them.

## Replication

* Every partition is replicated in kafka.Rule is Kafka gives a gurantee that your data in a partition will be there until N-1 brokers fails.
* So that means says i have 3 brokers ,then my partitions will be there until 2 brokers goes down.To provide this gurantee,the number of replication factor per partition that you can provide can be only upto to \(&lt;=\) Number of brokers.You cannot have two brokers and 3 replication factor for a partition/topic.
* Every replica is classified into **leader** and** follower .**
* **leader keeps track of which all followers are in sync and out ofsync**.Way it does this is each follower broker sends Fetch request to the leader same like how consumer does.it always asks the request in order ie if msg 1 and 2 is replicated only then it will ask for msg 3.This way the leader knows where the follower is based on its request.
* Leader makes the follower to be out of sync if it does NOt send a fetch request in 10 seconds or if it sends a fetch request but its not update till the last offset .
* **followers which are out of sync will NOT be eligible to become leader if the leader broker goes down. **time after which a follower is considered out of sync as above is based on value **replica.lag.time.max.ms**

## Requests

* We have 3 main types of requests : Produce ,Fetch and Metadata.
* Each broker has a metadata cache which has information about each topics ,its correspoding partitions along  its leaders and followers .
* A client will connect to any broker that you have given in your code\(bootstrap.servers\) and it will send a Metadata Request,now the client will know which is the Leader for a given partition and it will send a produce or fetch request based on that.Next it will use this info to send requests.
* Now the client will keep the metadata information upto date by resending the request for every **metadata.max.age.ms** or when a client sends fetch/poduce request but gets a exception saying "Not a leader" from the broker.
* During Producer request ,if acks=-1/all ,then once the leader gets the data ,it will store the ack request in a queue called "purgatory" untill all the followers are upto date.
* Fetch : this is sent by consumer clients or by follower replias brokers.It will usually be a requests aksing data from multiple topics partitions offsets, like from topic t1 ,partition 0 ,offset 10, from topic t2 partition 1 offset 12 \(with assumption that this broker is the leader for these partitions\).
* Clients can specify the min and max amount of data it can recieve\(look at consumer chapter fo configs to control this\)
* **Can Producer also control amount of information it can send?**I know broker has max.message.bytes to say how much max information per message can it be.
* If a client sends a fetch request for a topic or partition which does not exists,it will get a error.



