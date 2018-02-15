* Kafka takes only bytes as input data .No data verification ie data that you send kafka as bytes are just stored in brokers and the when asked is given to consumers,no data transformation and hence NO CPU usage at broker level.
* Problem with this byte approach is if Kafka Producers changes schema and writes data to broker then the corresponding consumer which reads it also needs to change else it will break.
* For this we will have SchemaRegistry : which needs to impose schema check,schema evoution and lightweigth.
* Basically we can write and read data in Avro directly into kafka but the problem is Avro will have coresponding schema also saved with the data for each record and to avoid this space consumption and delay ,we can save schema of a message in SchemaRegistry and then SchemaRegistry will return a unique id for that Schema which will be saved along with data by Producer.Now when consumer reads it it will refer to schema registry and get the correspoding schema for the message based on the id.All the interaction with SchemaRegistry is done at Serialization level.
* Avro is the only one supported in SchemaRegistry.Each message in Kafka needs to be independent and self sufficient and hence the benfit we ideally get from Columnar format like Parquet/ORC is not valid in Kafka.


