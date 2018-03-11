Some basic guarantess from Kafka:

* If two messages A and B are written to a same topic and same partition and A is written first and then B ,then offset of B is always greater than that of A.
* Messages are considered "committed" when all its sync are updated.Consumers can read from them only once all are in sync.
* Messages that are committed will NOT be lost until one replica remains.
* Consumers can only read messages that are "committed"\(See 2nd Point\).



A replica is said to be "in-sync"  when :

* The broker which as that replica is active and connected to zookeeper ie sending heartbeats and has /brokers/id/&lt;its\_id&gt;.
* its has been sending fetch request regularly\(10 secs\) to leader.Configurable.See Internals -1.
* It has sent the fetch request for the latest message within 10 seconds.Configurable.See Internals -1.

Once the replic becomes out of sync ,then consumer and producer does NOT take that replica into account.Nor it that taken into account while selecting the leader if the existing leader broker goes down.

**We can change the replication factor of a topic even after it has been created **.

Kafka in terms of partition replication ,will be rack aware when we set **broker.rack ** = &lt;rack-id&gt; config.

The algorithm used to assign replicas to brokers ensures that the number of leaders per broker will be constant, regardless of how brokers are distributed across racks. This ensures balanced throughput.However if racks are assigned different numbers of brokers, the assignment of replicas will not be even. _Racks with fewer brokers will get more replicas, meaning they will use more storage and put more resources into replication. Hence it is sensible to configure an equal number of brokers per rack._







