# [Book] [Juha Hinkula] Microservices with Spring Boot and Spring Cloud [ENG, 2021]

<br/>

## Chapter 04. Deploying Our Microservices Using Docker

<br/>

```
$ cd apps/Chapter04/
$ ./gradlew build
$ docker-compose build
```

<br/>

```
$ docker images | grep chapter04
```

<br/>

```
chapter04_recommendation      latest              4cc3655a6990   16 seconds ago   290MB
chapter04_product             latest              4541cd155f0d   20 seconds ago   290MB
chapter04_product-composite   latest              8a5c5e313258   23 seconds ago   290MB
chapter04_review              latest              7f416dede52a   27 seconds ago   290MB
```

<br/>

```
$ docker-compose up -d
```

<br/>

```
$ curl localhost:8080/product-composite/123 -s | jq .
```

<br/>

**response:**  

```
{
  "productId": 123,
  "name": "name-123",
  "weight": 123,
  "recommendations": [
    {
      "recommendationId": 1,
      "author": "Author 1",
      "rate": 1
    },
    {
      "recommendationId": 2,
      "author": "Author 2",
      "rate": 2
    },
    {
      "recommendationId": 3,
      "author": "Author 3",
      "rate": 3
    }
  ],
  "reviews": [
    {
      "reviewId": 1,
      "author": "Author 1",
      "subject": "Subject 1"
    },
    {
      "reviewId": 2,
      "author": "Author 2",
      "subject": "Subject 2"
    },
    {
      "reviewId": 3,
      "author": "Author 3",
      "subject": "Subject 3"
    }
  ],
  "serviceAddresses": {
    "cmp": "1f571512e69a/172.22.0.2:8080",
    "pro": "7035d737727d/172.22.0.4:8080",
    "rev": "62dc69901316/172.22.0.5:8080",
    "rec": "0b885554947d/172.22.0.3:8080"
  }
}
```


<br/>

```
$ ./test-em-all.bash
```

<br/>


```
Start Tests: Tue 17 May 2022 11:19:02 AM MSK
HOST=localhost
PORT=8080
Wait for: curl http://localhost:8080/product-composite/1... DONE, continues...
Test OK (HTTP Code: 200)
Test OK (actual value: 1)
Test OK (actual value: 3)
Test OK (actual value: 3)
Test OK (HTTP Code: 404, {"timestamp":"2022-05-17T08:19:02.288371158Z","path":"/product-composite/13","message":"No product found for productId: 13","status":404,"error":"Not Found"})
Test OK (actual value: No product found for productId: 13)
Test OK (HTTP Code: 200)
Test OK (actual value: 113)
Test OK (actual value: 0)
Test OK (actual value: 3)
Test OK (HTTP Code: 200)
Test OK (actual value: 213)
Test OK (actual value: 3)
Test OK (actual value: 0)
Test OK (HTTP Code: 422, {"timestamp":"2022-05-17T08:19:02.589842218Z","path":"/product-composite/-1","message":"Invalid productId: -1","status":422,"error":"Unprocessable Entity"})
Test OK (actual value: "Invalid productId: -1")
Test OK (HTTP Code: 400, {"timestamp":"2022-05-17T08:19:02.644+00:00","path":"/product-composite/invalidProductId","status":400,"error":"Bad Request","message":"Type mismatch.","requestId":"aa7e8fbb-1"})
Test OK (actual value: "Type mismatch.")
End, all tests OK: Tue 17 May 2022 11:19:02 AM MSK
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