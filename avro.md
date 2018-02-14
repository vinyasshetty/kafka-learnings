* This is a format where data is stored is serialized and compressed way usually in binary but can be json also.Every record in that datafile file has it correspoding schema also in json format.
* Many languages can read the data since it needs to read binary and schema via json.
* Avro is really helpful when it comes to schema evolution ie we can have some columns \(of union type\) which can be optional while writing \(ends up as null\) and also while reading we can refer to non existing columns which will just give us null.
* This has NO delimiter
* CSV\(No schema\) VS RDBMS\(Tough to move around data between systems,schema evolution possible but has limitations\) VS JSON\(NO Schema ie datatype check\) VS AVRO
* Avro Primitive Types =&gt; null,int,long,boolean,float,double,string,bytes



## Avro Record Schemas:

* Name :Name of the schema
* Namespace : This is like package name
* Doc :Documentation
* Fields : This is a Array ,conatining below: 

          +  Name  









