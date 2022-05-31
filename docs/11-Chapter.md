# [Book] [Magnus Larsson] Microservices with Spring Boot and Spring Cloud [ENG, 2021]

<br/>

## Chapter 11. Securing Access to APIs

<br/>

![Application](/img/ch11-pic01.png?raw=true)

<br/>

The self-signed certificate is created with the following command:

The source code comes with a sample certificate file, so you don't need to run this command to run the following examples.

```
$ keytool -genkeypair -alias localhost -keyalg RSA -keysize 2048
-storetype PKCS12 -keystore edge.p12 -validity 3650
```

The certificate file created, edge.p12 , is placed in the gateway projects folder, src/main/resources/keystore . This means that the certificate file will be placed in the .jar file when it is built and will be available on the classpath at runtime at keystore/edge.p12.

Providing certificates using the classpath is sufficient during development, but not applicable to other environments, for example, a production environment.


<br/>

### Testing with the local authorization server

<br/>

```
$ cd apps/Chapter11/
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
// Get an access token for the writer client
// OK!
https://u:p@localhost:8443/eureka/web
https://u:p@localhost:8443/eureka/apps
```

<br/>

```
$ curl -H "accept:application/json" https://u:p@localhost:8443/eureka/api/apps -ks | jq -r .applications.application[].instance[].instanceId
```

<br/>

**response:**

<br/>

```
e11fab100c00:gateway:8443
49dd77a41f17:auth-server:9999
b3ed849a9fa4:product-composite:8080
f17d5b865bbd:product:8080
59481cc98263:review:8080
9c3c2b6c1532:recommendation:8080
```

<br/>

```
// READER
$ curl -k https://reader:secret@localhost:8443/oauth2/token -d grant_type=client_credentials -s | jq .
```

<br/>

```
// WRITER
$ curl -k https://writer:secret@localhost:8443/oauth2/token -d grant_type=client_credentials -s | jq .
```

<br/>

**response:**

```
{
  "access_token": "eyJraW...m0A",
  "scope": "product:write openid product:read",
  "token_type": "Bearer",
  "expires_in": "3599"
}
```

<br/>

### Acquiring access tokens using the authorization code grant flow

```
// u / p
https://localhost:8443/oauth2/authorize?response_type=code&client_id=reader&redirect_uri=https://my.redirect.uri&scope=product:read&state=35725
```

<br/>

**Will redirect on:**

```
https://my.redirect.uri/?code=vTCgG8YxakUgCcqUK0ocwjFuTcwnM-MUA1UTsM5IG9Ej6PQgWx5MhHbwbes-x7XXpjdOY5KuaizprmNV0eryPbi_v-Z9TZOgZ9lPrjMmooScVj1IW4TVeHuUuRZCiwMg&state=35725
```

<br/>

```
$ export CODE=<AUTH_TOKEN>
```

<br/>

```
// READER
$ curl -k https://reader:secret@localhost:8443/oauth2/token \
-d grant_type=authorization_code \
-d client_id=reader \
-d redirect_uri=https://my.redirect.uri \
-d code=$CODE -s | jq .
```

<br/>

```
{
  "access_token": "eyJr...TDg",
  "refresh_token": "uDF...fR6q_yy",
  "scope": "product:read",
  "token_type": "Bearer",
  "expires_in": "3599"
}
```

<br/>

```
// To get an authorization code for the writer client
https://localhost:8443/oauth2/authorize?response_type=code&client_id=writer&redirect_uri=https://my.redirect.uri&scope=product:read+product:write&state=72489
```

<br/>

```
// WRITER
$ curl -k https://writer:secret@localhost:8443/oauth2/token \
-d grant_type=authorization_code \
-d client_id=writer \
-d redirect_uri=https://my.redirect.uri \
-d code=$CODE -s | jq .
```

<br/>

### Calling protected APIs using access tokens

<br/>

```
// READER
$ curl -k https://reader:secret@localhost:8443/oauth2/token -d grant_type=client_credentials -s | jq .
```

<br/>

```
$ READER_ACCESS_TOKEN=$(curl -k https://reader:secret@localhost:8443/oauth2/token -d grant_type=client_credentials -s | jq .access_token -r)

$ ACCESS_TOKEN=${READER_ACCESS_TOKEN}

$ echo ${ACCESS_TOKEN}

$ curl https://localhost:8443/product-composite/1 -k -H "Authorization: Bearer $ACCESS_TOKEN" -i
```

<br/>

**response:**

```
HTTP/1.1 200 OK
Content-Type: application/json
Cache-Control: no-cache, no-store, max-age=0, must-revalidate
Pragma: no-cache
Expires: 0
X-Content-Type-Options: nosniff
Strict-Transport-Security: max-age=31536000 ; includeSubDomains
X-Frame-Options: DENY
X-XSS-Protection: 1 ; mode=block
Referrer-Policy: no-referrer
content-length: 714

{"productId":1,"name":"product name C","weight":300,"recommendations":[{"recommendationId":1,"author":"author 1","rate":1,"content":"content 1"},{"recommendationId":2,"author":"author 2","rate":2,"content":"content 2"},{"recommendationId":3,"author":"author 3","rate":3,"content":"content 3"}],"reviews":[{"reviewId":1,"author":"author 1","subject":"subject 1","content":"content 1"},{"reviewId":2,"author":"author 2","subject":"subject 2","content":"content 2"},{"reviewId":3,"author":"author 3","subject":"subject 3","content":"content 3"}],"serviceAddresses":{"cmp":"b3ed849a9fa4/172.22.0.9:8080","pro":"f17d5b865bbd/172.22.0.10:8080","rev":"59481cc98263/172.22.0.8:8080","rec":"9c3c2b6c1532/172.22.0.11:8080"}}
```

<br/>

### Testing Swagger UI with OAuth 2.0

<br/>

```
https://localhost:8443/openapi/swagger-ui.html
```

Authorize -> Select All -> Autrorize -> u / p


<br/>

### [SKIPPED] Testing with an external OpenID Connect provider

https://auth0.com/

https://openid.net/developers/certified/


<br/>

```
$ docker-compose logs product-composite | grep "Authorization info"
```

<br/>

```
$ docker-compose down
```


<br/>

### Replacing a self-signed certificate at runtime

To replace the certificate packaged in the .jar file, perform the following steps:


<br/>

```
$ cd apps/Chapter11/
$ mkdir keystore
$ keytool -genkeypair -alias localhost -keyalg RSA -keysize 2048 -storetype PKCS12 -keystore keystore/edge-test.p12 -validity 3650
```

<br/>

```
// docker-compose restart gateway does not take changes in docker-compose.yml
$ docker-compose up -d --scale gateway=0
$ docker-compose up -d --scale gateway=1
```


<br/><br/>

---

<br/>

**Marley**

Any questions in english: <a href="https://javadev.org/chat/">Telegram Chat</a>  
Любые вопросы на русском: <a href="https://javadev.ru/chat/">Телеграм чат</a>