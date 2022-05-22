# [Book] [Magnus Larsson] Microservices with Spring Boot and Spring Cloud [ENG, 2021]

<br/>

## Chapter 07. Developing Reactive Microservices

<br/>

### Developing event-driven asynchronous services

<br/>

![Application](/img/ch07-pic01.png?raw=true)


<br/>

```
$ cd apps/Chapter07/
```

<br/>

### Using RabbitMQ without using partitions


<br/>

```
// RUN
$ ./gradlew build && docker-compose build && docker-compose up -d
```

<br/>

```
// TEST
$ curl -s localhost:8080/actuator/health | jq -r .status
```

<br/>

**returns:**

```
UP
```

<br/>

```
$ body='{"productId":1,"name":"product name C","weight":300, 
"recommendations":[
  {"recommendationId":1,"author":"author 1","rate":1,"content":"content 1"},
  {"recommendationId":2,"author":"author 2","rate":2,"content":"content 2"},
  {"recommendationId":3,"author":"author 3","rate":3,"content":"content 3"}
], 
"reviews":[
  {"reviewId":1,"author":"author 1","subject":"subject 1","content":"content 1"}, 
  {"reviewId":2,"author":"author 2","subject":"subject 2","content":"content 2"}, 
  {"reviewId":3,"author":"author 3","subject":"subject 3","content":"content 3"}
]}'


// OK!
$ curl -X POST localhost:8080/product-composite -H "Content-Type: application/json" --data "$body"
```

<br/>

```
// guest / guest
http://localhost:15672/#/queues
```

<br/>

![Application](/img/ch07-pic02.png?raw=true)


<br/>

![Application](/img/ch07-pic03.png?raw=true)

<br/>

```
$ curl -s localhost:8080/product-composite/1 | jq
```

<br/>

**response:**

```
{
  "productId": 1,
  "name": "product name C",
  "weight": 300,
  "recommendations": [
    {
      "recommendationId": 1,
      "author": "author 1",
      "rate": 1,
      "content": "content 1"
    },
    {
      "recommendationId": 2,
      "author": "author 2",
      "rate": 2,
      "content": "content 2"
    },
    {
      "recommendationId": 3,
      "author": "author 3",
      "rate": 3,
      "content": "content 3"
    }
  ],
  "reviews": [
    {
      "reviewId": 1,
      "author": "author 1",
      "subject": "subject 1",
      "content": "content 1"
    },
    {
      "reviewId": 2,
      "author": "author 2",
      "subject": "subject 2",
      "content": "content 2"
    },
    {
      "reviewId": 3,
      "author": "author 3",
      "subject": "subject 3",
      "content": "content 3"
    }
  ],
  "serviceAddresses": {
    "cmp": "e9f0835bfae7/172.22.0.7:8080",
    "pro": "4b0c9db01ac5/172.22.0.5:8080",
    "rev": "edc1b6fb8a8d/172.22.0.8:8080",
    "rec": "13e245499dc7/172.22.0.6:8080"
  }
}
```

<br/>

```
$ curl -X DELETE localhost:8080/product-composite/1
```

<br/>

```
$ curl -s localhost:8080/product-composite/1 | jq
```

<br/>

```
{
  "timestamp": "2022-05-21T00:09:03.365871261Z",
  "path": "/product-composite/1",
  "message": "No product found for productId: 1",
  "error": "Not Found",
  "status": 404
}
```

<br/>


```
$ docker-compose down
```


<br/>

### Using RabbitMQ with partitions

<br/>

```
$ export COMPOSE_FILE=docker-compose-partitions.yml
$ docker-compose build && docker-compose up -d
```

<br/>

```
$ docker-compose down
$ unset COMPOSE_FILE
```

<br/>

### Using Kafka with two partitions per topic

<br/>

```
$ export COMPOSE_FILE=docker-compose-kafka.yml
$ docker-compose build && docker-compose up -d
```

<br/>

```
$ curl product 1 and 2
```

<br/>

```
$ docker-compose exec kafka /opt/kafka/bin/kafka-topics.sh --zookeeper zookeeper --list
```

<br/>

**response:**

```
__consumer_offsets
error.products.productsGroup
error.recommendations.recommendationsGroup
error.reviews.reviewsGroup
products
recommendations
reviews
```

<br/>

```
$ docker-compose exec kafka /opt/kafka/bin/kafka-topics.sh --describe --zookeeper zookeeper --topic products
```

<br/>

**response:**

```
Topic: products	PartitionCount: 2	ReplicationFactor: 1	Configs: 
	Topic: products	Partition: 0	Leader: 1001	Replicas: 1001	Isr: 1001
	Topic: products	Partition: 1	Leader: 1001	Replicas: 1001	Isr: 1001
```

<br/>

```
// To see all the messages in a specific topic
$ docker-compose exec kafka /opt/kafka/bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic products --from-beginning --timeout-ms 1000
```

<br/>

**response:**

```
{"eventType":"CREATE","key":2,"data":{"productId":2,"name":"product name C","weight":300,"serviceAddress":null},"eventCreatedAt":"2022-05-21T00:27:59.790710261Z"}
{"eventType":"CREATE","key":4,"data":{"productId":4,"name":"product name C","weight":300,"serviceAddress":null},"eventCreatedAt":"2022-05-21T00:28:25.438645616Z"}
{"eventType":"CREATE","key":1,"data":{"productId":1,"name":"product name C","weight":300,"serviceAddress":null},"eventCreatedAt":"2022-05-21T00:22:05.101701016Z"}
{"eventType":"CREATE","key":3,"data":{"productId":3,"name":"product name C","weight":300,"serviceAddress":null},"eventCreatedAt":"2022-05-21T00:28:13.054835524Z"}
[2022-05-21 00:28:49,867] ERROR Error processing message, terminating consumer process:  (kafka.tools.ConsoleConsumer$)
org.apache.kafka.common.errors.TimeoutException
Processed a total of 4 messages
```

<br/>

```
// To see all the messages in a specific partition
$ docker-compose exec kafka /opt/kafka/bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic products --from-beginning --timeout-ms 1000 --partition 1
```

<br/>

**response:**

```
{"eventType":"CREATE","key":1,"data":{"productId":1,"name":"product name C","weight":300,"serviceAddress":null},"eventCreatedAt":"2022-05-21T00:22:05.101701016Z"}
{"eventType":"CREATE","key":3,"data":{"productId":3,"name":"product name C","weight":300,"serviceAddress":null},"eventCreatedAt":"2022-05-21T00:28:13.054835524Z"}
[2022-05-21 00:29:39,579] ERROR Error processing message, terminating consumer process:  (kafka.tools.ConsoleConsumer$)
org.apache.kafka.common.errors.TimeoutException
Processed a total of 2 messages
```

<br/>

```
$ docker-compose down
$ unset COMPOSE_FILE
```

<br/>

### Running automated tests of the reactive microservice landscape

<br/>

```
// Run the tests using the default Docker Compose file
$ unset COMPOSE_FILE
$ ./test-em-all.bash start stop
```

<br/>

```
// Run the tests for RabbitMQ with two partitions per topic
$ export COMPOSE_FILE=docker-compose-partitions.yml
$ ./test-em-all.bash start stop
$ unset COMPOSE_FILE
```

<br/>

```
// Run the tests with Kafka and two partitions per topic
$ export COMPOSE_FILE=docker-compose-kafka.yml
$ ./test-em-all.bash start stop
$ unset COMPOSE_FILE
```

<br/><br/>

---

<br/>

**Marley**

Any questions in english: <a href="https://javadev.org/chat/">Telegram Chat</a>  
Любые вопросы на русском: <a href="https://javadev.ru/chat/">Телеграм чат</a>