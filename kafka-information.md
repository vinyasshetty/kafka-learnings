# **Topics**

* Topics is a append only commit-log.
* Messages/Events that are sent to a topic can only be appended.ie data once written to a topic is immutable.
* Data written to a topic should be of same schema

# **Broker **

* Single Kafka Server is called a Broker.
* Broker receives messages from producers,then assigns them a offset and commits the message to storage on disk.
* It also services consumers who want to fetch data from a given topic and from a given partition and a given sequence.
* If Consumers and Producers wants to read/write messages then they always talks to the Broker which is the leader of the partitions.Replicas of the partitions are just used when the Broker which is the leader for a given partition fails.
* In config/server.properties we have the log.dir which determines where the data that you write into kafka will be saved.
* Example : log.dirs=/tmp/kafka-logs-1 ,so once u create a topic then we will have a directory called **/tmp/kafka-logs-1/&lt;topicname&gt;-&lt;partition\_count&gt;/** will be created.



