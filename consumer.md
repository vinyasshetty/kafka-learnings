## Consumer

* Maximum number of consumers you can have in a consumer groups should be equal to the number of partitions of a given topic.
* In other words,two or more partitions in a topic cannot send data to a single consumer within a consumer group.
* Data written to a topic can be read by multiple consumers,so make sure each our downstream app have thier own consumer group ,so that they can read the entirety of data from the topic.
* If we add a new consumer to a consumer group then this new consumer will get some partition allocated and the old consumer which used to get data from this partition will NOT get it anymore.Same happens when a consumer goes down in a consumer group or when admin adds a partition to a topic\(NOT a good thing ,due to chnages in partitioining of keys\).
* This process is called "rebalancing",during this time whole consumer group becomes unavailable ,so we need to be careful  about these rebalancing.
* Consumers keep sending a regular heart beats to a consumer group co-ordinator broker.This may be different for different consumer group.It sends heart beat when it polls to get records and commits when it had read the message.
* If a consumer does NOT send a heartbeat long enough then the group co-ordinator will decide that consumer is dead and retrigger a rebalancing.We can control the frequency of heartbeats and duration after which group co-ordinator decides that consumer is dead.
* Consumer which wants to join a group sends a JoinGroup request to the GC\(Group Co-ordinator\) broker,the first consumer to join the group will be the leader and GC will send all ther consumers which info who want to join the group,then the leader consumer using a partitioing policy allocates partitions to the consumers and sends this info to the GC.Then GC sends only the required info a individual consumer saying from whicj partition they need to read from.ONly the GC and Consumer leader has the whole assignment list.This process will be done everytime when there is a rebalancing.&lt;Q**uestion** : So can the Consumer leader change everytime there is a relanacing??&gt;



