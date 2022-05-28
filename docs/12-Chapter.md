# [Book] [Magnus Larsson] Microservices with Spring Boot and Spring Cloud [ENG, 2021]

<br/>

## Chapter 12. Centralized Configuration

<br/>

![Application](/img/ch12-pic01.png?raw=true)


<br/>

```
$ cd apps/Chapter12/
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
$ curl https://dev-usr:dev-pwd@localhost:8443/config/product/docker -ks | jq .
```

<br/>

**response:**

```
{
  "name": "product",
  "profiles": [
    "docker"
  ],
  "label": null,
  "version": null,
  "state": null,
  "propertySources": [
    {
      "name": "Config resource 'file [/config-repo/product.yml]' via location 'file:/config-repo/' (document #1)",
      "source": {
        "spring.config.activate.on-profile": "docker",
        "server.port": 8080,
        "spring.data.mongodb.host": "mongodb"
      }
    },
    {
      "name": "Config resource 'file [/config-repo/product.yml]' via location 'file:/config-repo/' (document #0)",
      "source": {
        "server.port": 7001,
        "server.error.include-message": "always",
        "spring.data.mongodb.host": "localhost",
        "spring.data.mongodb.port": 27017,
        "spring.data.mongodb.database": "product-db",
        "spring.cloud.function.definition": "messageProcessor",
        "spring.cloud.stream.default.contentType": "application/json",
        "spring.cloud.stream.bindings.messageProcessor-in-0.destination": "products",
        "spring.cloud.stream.bindings.messageProcessor-in-0.group": "productsGroup",
        "spring.cloud.stream.bindings.messageProcessor-in-0.consumer.maxAttempts": 3,
        "spring.cloud.stream.bindings.messageProcessor-in-0.consumer.backOffInitialInterval": 500,
        "spring.cloud.stream.bindings.messageProcessor-in-0.consumer.backOffMaxInterval": 1000,
        "spring.cloud.stream.bindings.messageProcessor-in-0.consumer.backOffMultiplier": 2,
        "spring.cloud.stream.rabbit.bindings.messageProcessor-in-0.consumer.autoBindDlq": true,
        "spring.cloud.stream.rabbit.bindings.messageProcessor-in-0.consumer.republishToDlq": true,
        "spring.cloud.stream.kafka.bindings.messageProcessor-in-0.consumer.enableDlq": true,
        "logging.level.root": "INFO",
        "logging.level.se.magnus": "DEBUG",
        "logging.level.org.springframework.data.mongodb.core.ReactiveMongoTemplate": "DEBUG"
      }
    },
    {
      "name": "Config resource 'file [/config-repo/application.yml]' via location 'file:/config-repo/' (document #1)",
      "source": {
        "spring.config.activate.on-profile": "docker",
        "spring.rabbitmq.host": "rabbitmq",
        "spring.cloud.stream.kafka.binder.brokers": "kafka",
        "app.eureka-server": "eureka",
        "app.auth-server": "auth-server"
      }
    },
    {
      "name": "Config resource 'file [/config-repo/application.yml]' via location 'file:/config-repo/' (document #0)",
      "source": {
        "app.eureka-username": "u",
        "app.eureka-server": "localhost",
        "app.auth-server": "localhost",
        "eureka.client.serviceUrl.defaultZone": "http://${app.eureka-username}:${app.eureka-password}@${app.eureka-server}:8761/eureka/",
        "eureka.client.initialInstanceInfoReplicationIntervalSeconds": 5,
        "eureka.client.registryFetchIntervalSeconds": 5,
        "eureka.instance.leaseRenewalIntervalInSeconds": 5,
        "eureka.instance.leaseExpirationDurationInSeconds": 5,
        "spring.rabbitmq.host": "127.0.0.1",
        "spring.rabbitmq.port": 5672,
        "spring.rabbitmq.username": "guest",
        "spring.cloud.stream.kafka.binder.brokers": "127.0.0.1",
        "spring.cloud.stream.kafka.binder.defaultBrokerPort": 9092,
        "spring.cloud.stream.defaultBinder": "rabbit",
        "management.endpoint.health.show-details": "ALWAYS",
        "management.endpoints.web.exposure.include": "*",
        "app.eureka-password": "p",
        "spring.rabbitmq.password": "guest"
      }
    }
  ]
}
```

<br/>

```
// ENCRYPT
$ curl -k https://dev-usr:dev-pwd@localhost:8443/config/encrypt --data-urlencode "hello world"
```

<br/>

```
// DECRYPT
$ curl -k https://dev-usr:dev-pwd@localhost:8443/config/decrypt -d 9eca39e823957f37f0f0f4d8b2c6c46cd49ef461d1cab20c65710823a8b412ce
```

<br/>

If you want to use an encrypted value in a configuration file, you need to prefix it with {cipher} and wrap it in '' .

<br/>

```
$ docker-compose down
```

<br/><br/>

---

<br/>

**Marley**

Any questions in english: <a href="https://javadev.org/chat/">Telegram Chat</a>  
Любые вопросы на русском: <a href="https://javadev.ru/chat/">Телеграм чат</a>