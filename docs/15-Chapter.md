# [Book] [Magnus Larsson] Microservices with Spring Boot and Spring Cloud [ENG, 2021]

<br/>

## Chapter 15. Introduction to Kubernetes

<br/>

### Run Minikube

<br/>

```
$ unset KUBECONFIG
```

<br/>

```
$ minikube start \
--profile=handson-spring-boot-cloud \
--memory=10240 \
--cpus=4 \
--disk-size=30g \
--kubernetes-version=v1.20.5 \
--driver=docker \
--ports=8080:80 --ports=8443:443 \
--ports=30080:30080 --ports=30443:30443
```

<br/>

```
$ minikube profile handson-spring-boot-cloud
$ minikube addons enable ingress
$ minikube addons enable metrics-server
```

<br/>

```
$ eval $(minikube docker-env)
$ docker pull mysql:5.7.32
$ docker pull mongo:4.4.2
$ docker pull rabbitmq:3.8.11-management
$ docker pull openzipkin/zipkin:2.23.2
```

<br/><br/>

---

<br/>

**Marley**

Any questions in english: <a href="https://javadev.org/chat/">Telegram Chat</a>  
Любые вопросы на русском: <a href="https://javadev.ru/chat/">Телеграм чат</a>