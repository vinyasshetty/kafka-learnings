# Need For Kafka

We have multiple upstreams sending data to multiple downstreams and this gets too confusing ,since one downstream might take data from multiple upstream and vice versa.

To overcome that we have Kafka which acts as a intermediate layer where all upstreams send data to Kafka and all downstreams read data from Kafka.

This causes complete decoupling ie Upstreams\(Publishers\) dont need to know anything about downstreams\(Subscribers\).

Kafka is a fast,distributed,highly scalable,higly available,publish-subscribe messaging system.

What Kafka Doent Do:

* No Encryption or authentication of/on data on Kafka.
* We need to write Consumer or Producers to move data out or into Kafka
* No data transformation.Data once written to Kafka is immutable.

**Kafka Compared to Other Messaging System:**

* Kafka provides the advantage of both Queue based and Publish-Subscribe based System.
* In Queue based system multiple consumers can read parts of the queue and thus providing with scalability but NO feature of multi-subscriber for one queue because once the data is read then its gone.
* In Publish Susbscribe mode ,we can have multiple consumers reading data but every subscriber gets the whole set and hence NO scalability.
* Kafka provides both these features using Consumer Group having multiple consumers in them.
* Mutiple Producers and Multiple Consumers can write/read from a topic.
* Easily scalable.
* Disk based retention provides great flexibility for saving data.



