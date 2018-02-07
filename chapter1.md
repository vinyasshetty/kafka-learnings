# Need For Kafka

We have multiple upstreams sending data to multiple downstreams and this gets too confusing ,since one downstream might take data from multiple upstream and vice versa.

To overcome that we have Kafka which acts as a intermediate layer where all upstreams send data to Kafka and all downstreams read data from Kafka.

Kafka is a fast,distributed,highly scalable,higly available,publish-subscribe messaging system.



What Kafka Doent Do:

* No Encryption or authentication .
* We need to write Consumer or Producers to move data out or into Kafka
* No data transformation.Data once written to Kafka is immutable.



