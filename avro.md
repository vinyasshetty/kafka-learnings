This is a format where data is stored is serialized way usually in binary but can be json also.Every record in that datafile file has it correspoding schema also in json format.

Avro is really helpful when it comes to schema evolution ie we can have some columns \(of union type\) which can be optional while writing \(ends up as null\) and also while reading we can refer to non existing columns which will just give us null.





