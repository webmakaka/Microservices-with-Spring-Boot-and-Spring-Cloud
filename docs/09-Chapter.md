# [Book] [Magnus Larsson] Microservices with Spring Boot and Spring Cloud [ENG, 2021]

<br/>

## Chapter 09. Adding Service Discovery Using Netflix Eureka

<br/>

```
$ cd apps/Chapter09/
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
// Launch two extra review microservice instances
$ docker-compose up -d --scale review=3
```

<br/>

```
http://localhost:8761/
```

<br/>

![Application](/img/ch09-pic01.png?raw=true)

<br/>

```
$ docker-compose logs review | grep Started
```

<br/>

```
$ curl -H "accept:application/json" localhost:8761/eureka/apps -s | jq -r .applications.application[].instance[].instanceId
```

<br/>

```
0b2d98dfb94f:product-composite:8080
34406669c17e:product:8080
540bd7d38888:review:8080
9498a90ff231:review:8080
b1c107f3d684:review:8080
e1ecafc5026f:recommendation:8080
```

<br/>


```
// IP address changes on each requests
$ curl localhost:8080/product-composite/1 -s | jq -r .serviceAddresses.rev
```

<br/>

```
b1c107f3d684/172.22.0.10:8080
```

<br/>

```
// Reesponse nothing for me
$ docker-compose logs review | grep getReviews
```

<br/>

### Scaling down

```
$ docker-compose up -d --scale review=2
```

<br/>

```
// We will wait no longer than 2 seconds for a
response
$ curl localhost:8080/product-composite/1 -m 2
```

<br/>

### Disruptive tests with the Eureka server

<br/>

```
// Stop the Eureka server and keep the two review instances up and
running
$ docker-compose up -d --scale review=2 --scale eureka=0
```

<br/>

```
$ curl localhost:8080/product-composite/1 -s | jq -r .serviceAddresses.rev
```

<br/>

```
// Terminate one of the two review instances
$ docker-compose up -d --scale review=1 --scale eureka=0
```

<br/>

```
$ curl localhost:8080/product-composite/1 -s | jq -r .serviceAddresses.rev
```

<br/>

### Starting up an extra instance of the product service

<br/>

```
$ docker-compose up -d --scale review=1 --scale eureka=0 --scale product=2
```

<br/>

```
// Something bad happened
// I am getting null
$ curl localhost:8080/product-composite/1 -s | jq -r .serviceAddresses.pro
```

<br/>

### Starting up the Eureka server again

<br/>

```
$ docker-compose up -d --scale review=1 --scale eureka=1 --scale product=2
```

<br/>

```
$ curl localhost:8080/product-composite/1 -s | jq -r .serviceAddresses
```

<br/>

**response:**

```
{
  "cmp": "0b2d98dfb94f/172.22.0.6:8080",
  "pro": "6c14798f8e61/172.22.0.9:8080",
  "rev": "83e0af55b4f6/172.22.0.4:8080",
  "rec": "e1ecafc5026f/172.22.0.7:8080"
}
```

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