Below are the 4 Gurantees :

1. Messages written to a given Topic Partition is in the same order as they are sent.
2. A single Consumer will read the message from the Topic partition in the order they are present in Topic Partition
3. If we have N replications,then rach partition is available until more then N-1 brokers go down.
4. If the messages with the same key are always sent to the same partition for a given topic provided we dont chnage the \# of partitions mid-way.



