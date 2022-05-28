# [Book] [Magnus Larsson] Microservices with Spring Boot and Spring Cloud [ENG, 2021]

<br/>

## Chapter 13. Improving Resilience Using Resilience4j


We will apply these mechanisms in one place, in calls from the product-composite service to the product service.

<br/>

![Application](/img/ch13-pic01.png?raw=true)


<br/>

```
$ cd apps/Chapter13/
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

```
// 200
$ curl -H "Authorization: Bearer $ACCESS_TOKEN" -k https://localhost:8443/product-composite/1 -w "%{http_code}\n" -o /dev/null -s
```

<br/>

```
// Expect it to respond with CLOSED
$ docker-compose exec product-composite curl -s http://product-composite:8080/actuator/health | jq -r .components.circuitBreakers.details.product.details.state
```

<br/>

### Forcing the circuit breaker to open when things go wrong


```
$ curl -H "Authorization: Bearer $ACCESS_TOKEN" -k https://localhost:8443/product-composite/1?delay=3 -s | jq .
```

<br/>

```
{
  "timestamp": "2022-05-28T11:47:28.520+00:00",
  "path": "/product-composite/1",
  "status": 500,
  "error": "Internal Server Error",
  "message": "Did not observe any item or terminal signal within 2000ms in 'onErrorResume' (and no fallback has been configured)",
  "requestId": "123ac88a-31"
}
```

Repeat request 4-5 times and you will receive:

```
{
  "productId": 1,
  "name": "Fallback product1",
  "weight": 1,
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
      "reviewId": 3,
      "author": "author 3",
      "subject": "subject 3",
      "content": "content 3"
    },
    {
      "reviewId": 2,
      "author": "author 2",
      "subject": "subject 2",
      "content": "content 2"
    }
  ],
  "serviceAddresses": {
    "cmp": "56f5724bff23/172.19.0.11:8080",
    "pro": "56f5724bff23/172.19.0.11:8080",
    "rev": "44a2c3c12703/172.19.0.10:8080",
    "rec": "49e52068278a/172.19.0.9:8080"
  }
}

```

<br/>

```
// Expect it to respond with HALF_OPEN
$ docker-compose exec product-composite curl -s http://product-composite:8080/actuator/health | jq -r .components.circuitBreakers.details.product.details.state
```

<br/>

### Closing the circuit breaker again


```
// Submit three normal requests to close the circuit breaker
$ curl -H "Authorization: Bearer $ACCESS_TOKEN" -k https://localhost:8443/product-composite/1 -w "%{http_code}\n" -o /dev/null -s
```

<br/>

```
// Expect it to respond with CLOSED
$ docker-compose exec product-composite curl -s http://product-composite:8080/actuator/health | jq -r .components.circuitBreakers.details.product.details.state
```

<br/>

```
$ docker-compose exec product-composite curl -s http://product-composite:8080/actuator/circuitbreakerevents/product/STATE_TRANSITION | jq -r '.circuitBreakerEvents[-3].stateTransition,.circuitBreakerEvents[-2].stateTransition, .circuitBreakerEvents[-1].stateTransition'
```

<br/>


### Trying out retries caused by random errors


```
$ time curl -H "Authorization: Bearer $ACCESS_TOKEN" -k https://localhost:8443/product-composite/1?faultPercent=25 -w "%{http_code}\n" -o /dev/null -s
```

<br/>

Repeat until response time of 1 second

<br/>

```
$ docker-compose exec product-composite curl -s http://product-composite:8080/actuator/retryevents | jq '.retryEvents[-2],.retryEvents[-1]'
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