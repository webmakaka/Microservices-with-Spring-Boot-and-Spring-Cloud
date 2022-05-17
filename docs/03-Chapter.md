# [Book] [Juha Hinkula] Microservices with Spring Boot and Spring Cloud [ENG, 2021]

<br/>

## Chapter 03. Creating a Set of Cooperating Microservices  

<br/>

```
Product composite service: 7000
Product service: 7001
Review service: 7002
Recommendation service: 7003
```

<br/>


```
$ cd apps/Chapter03/
$ ./gradlew build
$ ./gradlew test
```

<br/>

```
$ {
  java -jar microservices/product-composite-service/build/libs/*.jar &
  java -jar microservices/product-service/build/libs/*.jar &
  java -jar microservices/recommendation-service/build/libs/*.jar &
  java -jar microservices/review-service/build/libs/*.jar &
}
```

<br/>

```
$ curl http://localhost:7000/product-composite/1 -s | jq .
```

<br/>

**response:**  

```
{
  "productId": 1,
  "name": "name-1",
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
    "cmp": "workstation/127.0.1.1:7000",
    "pro": "workstation/127.0.1.1:7001",
    "rev": "workstation/127.0.1.1:7003",
    "rec": "workstation/127.0.1.1:7002"
  }
}
```


<br/>

```
$ ./test-em-all.bash
```

<br/>


```
HOST=localhost
PORT=7000
Test OK (HTTP Code: 200)
Test OK (actual value: 1)
Test OK (actual value: 3)
Test OK (actual value: 3)
Test OK (HTTP Code: 404, {"timestamp":"2022-05-17T06:30:46.355371098+03:00","path":"/product-composite/13","message":"No product found for productId: 13","error":"Not Found","status":404})
Test OK (actual value: No product found for productId: 13)
Test OK (HTTP Code: 200)
Test OK (actual value: 113)
Test OK (actual value: 0)
Test OK (actual value: 3)
Test OK (HTTP Code: 200)
Test OK (actual value: 213)
Test OK (actual value: 3)
Test OK (actual value: 0)
Test OK (HTTP Code: 422, {"timestamp":"2022-05-17T06:30:46.649201665+03:00","path":"/product-composite/-1","message":"Invalid productId: -1","error":"Unprocessable Entity","status":422})
Test OK (actual value: "Invalid productId: -1")
Test OK (HTTP Code: 400, {"timestamp":"2022-05-17T03:30:46.703+00:00","path":"/product-composite/invalidProductId","status":400,"error":"Bad Request","message":"Type mismatch.","requestId":"24e61f36-1"})
Test OK (actual value: "Type mismatch.")
End, all tests OK: Tue 17 May 2022 06:30:46 AM MSK

```

<br/>

```
$ kill $(jobs -p)
```

<br/><br/>

---

<br/>

**Marley**

Any questions in english: <a href="https://javadev.org/chat/">Telegram Chat</a>  
Любые вопросы на русском: <a href="https://javadev.ru/chat/">Телеграм чат</a>