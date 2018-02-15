* This is a format where data is stored is serialized and compressed way usually in binary but can be json also.Every record in that datafile file has it correspoding schema also in json format.
* Many languages can read the data since it needs to read binary and schema via json.
* Avro is really helpful when it comes to schema evolution ie we can have some columns \(of union type\) which can be optional while writing \(ends up as null\) and also while reading we can refer to non existing columns which will just give us null.
* This has NO delimiter
* CSV\(No schema\) VS RDBMS\(Tough to move around data between systems,schema evolution possible but has limitations\) VS JSON\(NO Schema ie datatype check\) VS AVRO
* Avro Primitive Types =&gt; null,int,long,boolean,float,double,string,bytes
* Complex Types =&gt; Record ,Enum,Arrays,Map,Union,Calling other schemas as types.

## Avro Record Schemas:

* **Type : "Record"**
* Name :Name of the schema
* Namespace : This is like package name
* aliases : Array of strings which name used instead of Name
* Doc :Documentation
* Fields : This is a Array ,conatining JSON objects having below info:

  * Name  
  * Type :Can have pritimive and/or complex types\(A JSON object defining a schema, or a JSON string naming a record definition \)
  * Doc
  * aliases : Array of strings which name used instead of Field Name
  * Defaults
  * Order** &lt;EDIT&gt; Need to understand more **

  `{`  
  `"type": "record",`  
  `"namespace": "com.example",`  
  `"name": "customer",`  
  `"doc": "this is some doc",`  
  `"fields": [`  
  `{ "name": "first_name", "type": "string", "doc": "This is firstname" },`  
  `{ "name": "automated_email", "type": "boolean", "doc": "This is a boolean","default" : true },`  
  `{"name" : "age","type":"int", "default": -1,doc : "This is age value"}`  
  `]`  
  `}`

## Avro Enum Schema

* Enums are the ones where we have fixed set of values.Example :
* **Symbols needs to JSON List of Strings and has to be unique.**
* **Symbols once set cannot be changed,can cause breakage if symbols are changed.**

`{`

`"type" : "enum",`

`"namespace" : ""`

`"name" : "sex",`

`"aliases" : ["Gender"]`

`"doc"  : "Determines the gender",`

`"symbols" : ["MALE","FEMALE","N/A"]`

`}`

## Avro Array Schema

This is for collections of data.items signify the datatype.

`{"type" : "array" , "items" : "string"}`

## Avro Map Schema

This is for key value pairs. Keys are always assumed to be String Type.

`{"type" : "map" , "values"  : "string"}`

## Avro Union Schema

This used to define default values. Represented as  \[  \]   . Example:

\["string","int","long"\]

**This is mainly used in =&gt;  "type" : \["null","string"\], "default" : null. Now the default should always be of the type of the first union.**

**Also you cannot have union within a union and Unions may not contain more than one schema with the same type, except for the named types record, fixed and enum.**

**It also makes this Field as Optional**

## Avro Fixed Type Schema

size determines the number of bytes required,

```
{"type": "fixed", "size": 16, "name": "md5","namespace" : "","aliases" : ["name1","name2"]}
```

Example Combined :

`[`

`{`

`"type" : "record",`

`"name" : "customer_info",`

`"namespace" : "com.pack",`

`"aliases" :["cust_info"],`

`"fields" : [`

`{"name" : "first_name","type" :"string","aliases": ["fname"]},`

`{"name" : "last_name","type" : ["null","string"],"default" : null},`

`{"name" : "age" , "type" : "int","default":-1 },`

`{"name" : "gender", "type" : "enum", "symbols" : ["MALE","FEMALE","N/A"]},`

`{"name" : "email", "type":"array","items" :"string" ,"default" : []},`

`{"name" : "sec_quest" , "type" : "map", "values" : "string","default":{"No quest" : ""} },`

`{"name" : "cust_address", "type" : "org.pack.cust_addr"}`

`]`

`},`

`{`

`"type" : "record",`

`"name" : "cust_addr",`

`"namespace" : "com.pack",`

`"fields ": [`

`{"name" : "street" , "type" : "string"},`

`{"name" : "city" ,"type": "string"},`

`{"name" : "zip","type":"string"},`

`{"name" : "staying" , "doc":"currently staying","type":"boolean","default":false}`

`]`

```
}
```

`]`

**&lt;EDIT &gt; LOGICAL TYPES =&gt; logicalType :Additional info on exitsing primitive types.**Different types are decimals\(bytes\),date\(int\),time-millis\(long\),timestamp-millis\(long\)



## **GenericRecord ** 

This is used to create avro objects from schema.Schema can be referenced from file or a string.

GitHub Link : &lt;EDIT&gt;

SETTING DATA:

If i dont set a mandatory field then it will throw a AvroRunTimeException error.But if i set a field which is not available in schema then it throws a NULLPOINTER EXCEPTION .These all are Runtime Exception.

GETTING DATA:

If you get a non existing column ,then u will get null.

