# [Book] [Magnus Larsson] Microservices with Spring Boot and Spring Cloud [ENG, 2021]

<br/>

## Chapter 14. Understanding Distributed Tracing


<br/>

![Application](/img/ch14-pic01.png?raw=true)

<br/>

we will store the trace information in memory.


<br/>

```
$ cd apps/Chapter14/
```

<br/>

```
$ ./gradlew build && docker-compose build
```

<br/>

```
$ ./test-em-all.bash start
```

<br/>

```
$ unset ACCESS_TOKEN
$ ACCESS_TOKEN=$(curl -k https://writer:secret@localhost:8443/oauth2/token -d grant_type=client_credentials -s | jq -r .access_token)
$ echo $ACCESS_TOKEN
```

<br/>

### Sending a successful API request

<br/>

```
// 200
$ curl -H "Authorization: Bearer $ACCESS_TOKEN" -k https://localhost:8443/product-composite/1 -w "%{http_code}\n" -o /dev/null -s
```

<br/>

http://localhost:9411/zipkin/

<br/>

+ -> serviceName and then gateway -> RUN QUERY


<br/>

### Sending an unsuccessful API request

<br/>

```
// 404
$ curl -H "Authorization: Bearer $ACCESS_TOKEN" -k https://localhost:8443/product-composite/12345 -w "%{http_code}\n" -o /dev/null -s
```


<br/>

### Sending an API request that triggers asynchronous processing

<br/>

```
// 202
$ curl -X DELETE -H "Authorization: Bearer $ACCESS_TOKEN" -k https://localhost:8443/product-composite/12345 -w "%{http_code}\n" -o /dev/null -s
```

<br/>

### Monitoring trace information passed to Zipkin in RabbitMQ

```
// guest / guest
http://localhost:15672/#/queues/%2F/zipkin
```

<br/>

```
$ docker-compose down
```

<br/>

### Using Kafka as a message broker

<br/>

```
$ export COMPOSE_FILE=docker-compose-kafka.yml
$ ./test-em-all.bash start
```

<br/>

**Do some requests:**

<br/>

```
$ docker-compose exec kafka /opt/kafka/bin/kafka-topics.sh --zookeeper zookeeper --list
```

<br/>

```
$ docker-compose exec kafka /opt/kafka/bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic zipkin --from-beginning --timeout-ms 1000
```

<br/>

```
$ docker-compose down
$ unset COMPOSE_FILE
```

<br/><br/>

---

<br/>

**Marley**

Any questions in english: <a href="https://javadev.org/chat/">Telegram Chat</a>  
Любые вопросы на русском: <a href="https://javadev.ru/chat/">Телеграм чат</a>