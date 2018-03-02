* Kafka takes only bytes as input data .No data verification ie data that you send kafka as bytes are just stored in brokers and the when asked is given to consumers,no data transformation and hence NO CPU usage at broker level.
* Problem with this byte approach is if Kafka Producers changes schema and writes data to broker then the corresponding consumer which reads it also needs to change else it will break.
* For this we will have SchemaRegistry : which needs to impose schema check,schema evoution and lightweigth.
* Basically we can write and read data in Avro directly into kafka but the problem is Avro will have coresponding schema also saved with the data for each record and to avoid this space consumption and delay ,we can save schema of a message in SchemaRegistry and then SchemaRegistry will return a unique id for that Schema which will be saved along with data by Producer.Now when consumer reads it it will refer to schema registry and get the correspoding schema for the message based on the id.All the interaction with SchemaRegistry is done at Serialization level.
* Avro is the only one supported in SchemaRegistry.Each message in Kafka needs to be independent and self sufficient and hence the benfit we ideally get from Columnar format like Parquet/ORC is not valid in Kafka.
* Schema registry has a option as what type of Evloution of SChema we need to support.Default is "backward".We can also set it None,Forward,Full.
* `kafka-avro-console-producer \`

  `--broker-list 127.0.0.1:9092 --topic test-avro \`

  `--property schema.registry.url=http://127.0.0.1:8081 \`

  `--property value.schema='{"type":"record","name":"myrecord","fields":[{"name":"f1","type":"string"}]}'`

* {"f1" : "value1"} =&gt;Works

* {"f2" : "value2"} =&gt; Error

* {"f1" :12} =&gt; Error

* {"f1" : "value2"}  =&gt; Works

* **We will have one Schema per Topic in Schema Registry and this can be evolved based on Avro Compatibility.In Schema Resgistry it will be named as **_**&lt;topic\_name&gt;-value**_** assuming the schema is for the value.**



