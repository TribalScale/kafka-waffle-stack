![media/Horse-and-Rider-1909-1910-.jpg](media/Horse-and-Rider-1909-1910-.jpg)
> *Horse and Rider 1909-1910 - Franz Kafka*

# 🥞 kafka-waffle-stack
Kafka stack with zookeeper, schema-registry, kafka-rest-proxy, kafka-connect, kafka-connect-ui, zoonavigator-web &amp; zoonavigator-api


## 🚂 What toys will you have?

- kafka
  * kafka-connect
  * kafka-connect-ui
  * kafka-topics-ui
  * kafka-rest-proxy

- zookeeper
  * zoonavigator-api
  * zoonavigator-web

- schema-registry
  * schema-registry-ui

- ksql

## 🖥 Nice UIs to play with

- Kafka topics UI
http://localhost:8000

- Schema registry UI
http://localhost:8001

- Kafka connect UI
http://localhost:8002

- Zoonavigator web
http://localhost:8003


## 🔧 Installation

0. ⭐️ Star the repo 😛
1. 📦 Install Docker - [instructions](https://docs.docker.com/install)
2. 🐑 Clone this repo
3. ⌨️ Run `docker-compose up`
4. ⏱ Wait... 😅
5. 🕹 Play and have fun

## Posting a message to kafka through kafka-rest-proxy

```sh
curl -X POST -H "Content-Type: application/vnd.kafka.json.v2+json" -H "Accept: application/vnd.kafka.v2+json" --data '{"records":[{"value":{"foo":"bar"}}]}' "http://localhost:8082/topics/jsontest"
```

> 📝 You can import 👆 into Postman to play with the values. `Import > plain text`

## Generating data into the kafka cluster
### simulate "user" data
```sh
docker run --network kafka-waffle-stack_default --rm --name datagen-users \
    confluentinc/ksql-examples:5.0.0 \
    ksql-datagen \
        bootstrap-server=kafka:9092 \
        quickstart=users \
        format=json \
        topic=users \
        maxInterval=100
```

### simulate "pageview" data
```sh
docker run --network kafka-waffle-stack_default --rm --name datagen-pageviews \
    confluentinc/ksql-examples:5.0.0 \
    ksql-datagen \
        bootstrap-server=kafka:9092 \
        quickstart=pageviews \
        format=delimited \
        topic=pageviews \
        maxInterval=500
```

## Connecting a ksql cli instance to ksql-server
```sh
docker run --network=kafka-waffle-stack_default -it confluentinc/cp-ksql-cli:5.0.0 http://ksql-server:8088
```

### Playing around with ksql
1. Run the "user" and "pageview" data generators
2. Run a ksql cli container connected to the ksql-server
3. Create a ksql stream: `CREATE STREAM pageviews_original (viewtime bigint, userid varchar, pageid varchar) WITH (kafka_topic='pageviews', value_format='DELIMITED');`
4. Create a ksql table: `CREATE TABLE users_original (registertime BIGINT, gender VARCHAR, regionid VARCHAR, userid VARCHAR) WITH (kafka_topic='users', value_format='JSON', key = 'userid');`
5. Run a one-off query to select data from a stream: `SELECT pageid FROM pageviews_original LIMIT 3;`
6. Create a persistent query that joins two topics together and writes to another topic:
```
      CREATE STREAM pageviews_enriched AS \
      SELECT users_original.userid AS userid, pageid, regionid, gender \
      FROM pageviews_original \
      LEFT JOIN users_original \
        ON pageviews_original.userid = users_original.userid;
```
7. View the results as they come in (CTRL+c to cancel, note that this does not end the underlying query): `SELECT * FROM pageviews_enriched;`

Check out [the source documentation](https://docs.confluent.io/current/ksql/docs/tutorials/basics-docker.html#ksql-quickstart-docker) for more fun examples!


## 🔍 Want to take a 👀 inside the machines?

```sh
docker-compose exec <name-of-the-container> /bin/bash
```

**For example**

```sh
docker-compose exec kafka /bin/bash
```

```sh
docker-compose exec kafka /bin/bash
root@kafka:/# ls
bin  boot  dev	etc  home  lib	lib64  media  mnt  opt	proc  root  run  sbin  srv  sys  tmp  usr  var
root@kafka:/# cd var/lib/kafka/data/
root@kafka:/var/lib/kafka/data# ls
__confluent.support.metrics-0  __consumer_offsets-2   __consumer_offsets-31  __consumer_offsets-43  cleaner-offset-checkpoint  docker-connect-offsets-19  docker-connect-offsets-9
__consumer_offsets-0	       __consumer_offsets-20  __consumer_offsets-32  __consumer_offsets-44  docker-connect-configs-0   docker-connect-offsets-2   docker-connect-status-0
__consumer_offsets-1	       __consumer_offsets-21  __consumer_offsets-33  __consumer_offsets-45  docker-connect-offsets-0   docker-connect-offsets-20  docker-connect-status-1
__consumer_offsets-10	       __consumer_offsets-22  __consumer_offsets-34  __consumer_offsets-46  docker-connect-offsets-1   docker-connect-offsets-21  docker-connect-status-2
__consumer_offsets-11	       __consumer_offsets-23  __consumer_offsets-35  __consumer_offsets-47  docker-connect-offsets-10  docker-connect-offsets-22  docker-connect-status-3
__consumer_offsets-12	       __consumer_offsets-24  __consumer_offsets-36  __consumer_offsets-48  docker-connect-offsets-11  docker-connect-offsets-23  docker-connect-status-4
__consumer_offsets-13	       __consumer_offsets-25  __consumer_offsets-37  __consumer_offsets-49  docker-connect-offsets-12  docker-connect-offsets-24  log-start-offset-checkpoint
__consumer_offsets-14	       __consumer_offsets-26  __consumer_offsets-38  __consumer_offsets-5   docker-connect-offsets-13  docker-connect-offsets-3   meta.properties
__consumer_offsets-15	       __consumer_offsets-27  __consumer_offsets-39  __consumer_offsets-6   docker-connect-offsets-14  docker-connect-offsets-4   recovery-point-offset-checkpoint
__consumer_offsets-16	       __consumer_offsets-28  __consumer_offsets-4   __consumer_offsets-7   docker-connect-offsets-15  docker-connect-offsets-5   replication-offset-checkpoint
__consumer_offsets-17	       __consumer_offsets-29  __consumer_offsets-40  __consumer_offsets-8   docker-connect-offsets-16  docker-connect-offsets-6
__consumer_offsets-18	       __consumer_offsets-3   __consumer_offsets-41  __consumer_offsets-9   docker-connect-offsets-17  docker-connect-offsets-7
__consumer_offsets-19	       __consumer_offsets-30  __consumer_offsets-42  _schemas-0		    docker-connect-offsets-18  docker-connect-offsets-8
```

## 💾  Volumes

The following volumes will be created inside the `./volumes` folder:

 - 📁 `./volumes/kafka`
 - 📁 `./volumes/zookeeper`


## Q&A
### ⚠️ Issues login into Zoonavigator?

When you visit Zoonavigator (http://localhost:8003) the first time, you will be prompted to enter the following details 👇

**Connection string:** `zookeeper:2181`
**no user / no password** just click Connect

### ⚠️ Unable to connect your application to Kafka broker?

You may need to add the following line to `/etc/hosts`, `C:\Windows\system32\drivers\etc\hosts`, etc:

`127.0.0.1 kafka`


## 💡 Ideas

- Use hotel for a nice UI
- kafka-stack cli that starts a UI with all the links


## 🙏 Thanks & Credits

- Illustration - [The art of Franz Kafka - The Drawings (1907-1917) OpenCulture](http://www.openculture.com/2014/02/the-art-of-franz-kafka-drawings-from-1907-1917.html)
