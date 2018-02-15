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
* Fields : This is a Array ,conatining below of below combinations:

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


