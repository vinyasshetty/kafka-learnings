* Kafka takes only bytes as input data .No data verification ie data that you send kafka as bytes are just stored in brokers and the when asked is given to consumers,no data transformation and hence NO CPU usage at broker level.
* Problem with this byte approach is if Kafka Producers changes schema and writes data to broker then the corresponding consumer which reads it also needs to change else it will break.
* For this we will have SchemaRegistry : which needs to impose schema check,schema evoution and lightweigth.
* Basically we can write and read data in Avro but the problem is Avro will have coresponding schema also saved with the data for each record and to avoid this space consumption and delay ,we can save schema of a message in SchemaRegistry and then assign a unique id which will be saved along with data.Now when consumer reads it it will refer to schema registry and get the correspoding schema for the message based on the id.All the interaction with SchemaRegistry is done at Serialization level.



