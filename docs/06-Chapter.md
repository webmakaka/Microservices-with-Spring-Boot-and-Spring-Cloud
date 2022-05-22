# [Book] [Magnus Larsson] Microservices with Spring Boot and Spring Cloud [ENG, 2021]

<br/>

## Chapter 06. Adding Persistence

The product and recommendation microservices will use Spring Data for MongoDB

The review microservice will use Spring Data for MySQL database.

<br/>

![Application](/img/ch06-pic01.png?raw=true)

<br/>

```
$ cd apps/Chapter06/
```

<br/>

```
// TESTS
$ ./gradlew microservices:product-service:test --tests PersistenceTests
$ ./gradlew microservices:recommendation-service:test --tests PersistenceTests
$ ./gradlew microservices:review-service:test --tests PersistenceTests
```

<br/>

```
// RUN
$ ./gradlew build && docker-compose build && docker-compose up
```

<br/>

```
// OK!
http://localhost:8080/openapi/swagger-ui.html
```

<br/>

```
1. Click on the ProductComposite service and the POST method to expand them
2. Click on the Try it out button and go down to the body field
3. Replace the default value, 0, of the productId field with 123456
4. Scroll down to the Execute button and click on it
5. Verify that the returned response code is 200
```

<br/>

```
// OK!
$ docker-compose exec mongodb mongo product-db --quiet --eval "db.products.find()"
```

<br/>

```
// OK!
$ docker-compose exec mongodb mongo recommendation-db --quiet --eval "db.recommendations.find()"
```

<br/>

```
// password is "pwd"
$ docker-compose exec mysql mysql -uuser -p review-db -e "select * from reviews"
```

<br/>

```
// OK!
$ ./test-em-all.bash
```


<br/><br/>

---

<br/>

**Marley**

Any questions in english: <a href="https://javadev.org/chat/">Telegram Chat</a>  
Любые вопросы на русском: <a href="https://javadev.ru/chat/">Телеграм чат</a>