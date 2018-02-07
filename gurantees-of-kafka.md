# Below are the 4 Gurantees :

1. Messages written to a given Topic Partition is in the same order as they are sent.
2. A single Consumer will read the message from the Topic partition in the order they are present in Topic Partition
3. If we have N replications,then rach partition is available until more then N-1 brokers go down.
4. If the messages with the same key are always sent to the same partition for a given topic provided we dont chnage the \# of partitions mid-way.

Kafka does NOT gurantee the message write/read order is maintained across the topics.

# Kafka Clients:

Kafka clients come in two flavours: producer and consumer. Each of these can be configured to different levels of consistency.

For a producer we have three choices. On each message we can \(1\) wait for all in sync replicas to acknowledge the message, \(2\) wait for only the leader to acknowledge the message, or \(3\) do not wait for acknowledgement. Each of these methods have their merits and drawbacks and it is up to the system implementer to decide on the appropriate strategy for their system based on factors like consistency and throughput.

On the consumer side, we can only ever read committed messages \(i.e., those that have been written to all in sync replicas\). Given that, we have three methods of providing consistency as a consumer: \(1\) receive each message at most once, \(2\) receive each message at least once, or \(3\) receive each message exactly once. Each of these scenarios deserves a discussion of its own.

For at most once message delivery, the consumer reads data from a partition, commits the offset that it has read, and then processes the message. If the consumer crashes between committing the offset and processing the message it will restart from the next offset without ever having processed the message. This would lead to potentially undesirable message loss.

A better alternative is at least once message delivery. For at least once delivery, the consumer reads data from a partition, processes the message, and then commits the offset of the message it has processed. In this case, the consumer could crash between processing the message and committing the offset and when the consumer restarts it will process the message again. This leads to duplicate messages in downstream systems but no data loss.System should be idempotent.

Exactly once delivery is guaranteed by having the consumer process a message and commit the output of the message along with the offset to a transactional system. If the consumer crashes it can re-read the last transaction committed and resume processing from there. This leads to no data loss and no data duplication. In practice however, exactly once delivery implies significantly decreasing the throughput of the system as each message and offset is committed as a transaction.

In practice most Kafka consumer applications choose at least once delivery because it offers the best trade-off between throughput and correctness. It would be up to downstream systems to handle duplicate messages in their own way.

