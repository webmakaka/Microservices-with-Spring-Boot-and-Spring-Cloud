# [Book] [Magnus Larsson] Microservices with Spring Boot and Spring Cloud [ENG, 2021]

<br/>

## Chapter 10. Using Spring Cloud Gateway to Hide Microservices behind an Edge Server


<br/>

![Application](/img/ch10-pic01.png?raw=true)

<br/>

```
$ cd apps/Chapter10/
```

<br/>

```
$ ./gradlew clean build && docker-compose build
```

<br/>

```
$ ./test-em-all.bash start
```

<br/>

```
$ docker-compose ps gateway eureka product-composite product
recommendation review
```

<br/>

**response:**

```
NAME                            COMMAND                  SERVICE             STATUS              PORTS
chapter10-eureka-1              "java org.springfram…"   eureka              running             8761/tcp
chapter10-gateway-1             "java org.springfram…"   gateway             running             0.0.0.0:8080->8080/tcp, :::8080->8080/tcp
chapter10-product-1             "java org.springfram…"   product             running             8080/tcp
chapter10-product-composite-1   "java org.springfram…"   product-composite   running             8080/tcp
```

<br/>

```
$ curl localhost:8080/actuator/gateway/routes -s | jq '.[] | {"\(.route_id)": "\(.uri)"}' | grep -v '{\|}'
```

<br/>

**response:**

```
  "product-composite": "lb://product-composite"
  "product-composite-swagger-ui": "lb://product-composite"
  "eureka-api": "http://eureka:8761"
  "eureka-web-start": "http://eureka:8761"
  "eureka-web-other": "http://eureka:8761"
  "host_route_200": "http://httpstat.us:80"
  "host_route_418": "http://httpstat.us:80"
  "host_route_501": "http://httpstat.us:80"
```

<br/>

### Calling the product composite API through the edge server

```
$ docker-compose logs -f --tail=0 gateway
```

<br/>

```
$ curl http://localhost:8080/product-composite/1
```

<br/>

```
// SWAGGER
// OK!
http://localhost:8080/openapi/swagger-ui.html
```


<br/>

### Calling Eureka through the edge server


```
$ curl -H "accept:application/json" \
localhost:8080/eureka/api/apps -s | \
jq -r .applications.application[].instance[].instanceId
```

<br/>

**response:**

```
717b5af69a79:gateway:8080
65393d07bd7e:product-composite:8080
6ca191a644d3:product:8080
93571c23a750:review:8080
ebf9ec487fd4:recommendation:8080
```

<br/>

```
// Eureka web page
// OK!
http://localhost:8080/eureka/web
```

<br/>

![Application](/img/ch10-pic02.png?raw=true)


<br/>

### Routing based on the host header

```
$ curl http://localhost:8080/headerrouting -H "Host: i.feel.lucky:8080"
```

<br/>

```
$ curl http://localhost:8080/headerrouting -H "Host: im.a.teapot:8080"
```

<br/>

```
$ curl http://localhost:8080/headerrouting
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