Kafka -&gt; config/server.properties has the broker.id which needs to be unique.If you want you can download kafka and start kafka using two copies of server.properties having different broker.id and logs location on your system.

This has the zookeeper address.Log directories etc.

For us to set up via docker .Set up Docker first.

Then on Terminal:

\# Docker for Mac &gt;= 1.12, Linux, Docker for Windows 10

`docker run --rm -it \`

`           -p 2181:2181 -p 3030:3030 -p 8081:8081 \`

`           -p 8082:8082 -p 8083:8083 -p 9092:9092 \`

`           -e ADV_HOST=127.0.0.1 \`

`           landoop/fast-data-dev`

\# Docker toolbox

`docker run --rm -it \`

`          -p 2181:2181 -p 3030:3030 -p 8081:8081 \`

`          -p 8082:8082 -p 8083:8083 -p 9092:9092 \`

`          -e ADV_HOST=192.168.99.100 \`

`          landoop/fast-data-dev`



\# Kafka command lines tools\(In a Separate Terminal\)

```
docker run --rm -it --net=host landoop/fast-data-dev bash
```

This will run with only one Broker.

