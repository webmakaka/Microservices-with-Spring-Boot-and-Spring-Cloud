# [Book] [Magnus Larsson] Microservices with Spring Boot and Spring Cloud [ENG, 2021]

<br/>

## Chapter 20. Monitoring Microservices


**Need to repeat steps from previous chapter**

### [Run Minikube](15-Chapter.md)

### [Setup istioctl](18-Chapter.md)

### [Install the cert-manager](18-Chapter.md)


<br/>

```
$ cd apps/Chapter20
$ eval $(minikube docker-env)
$ ./gradlew build && docker-compose build
```

<br/>

```
$ kubectl apply -f kubernetes/hands-on-namespace.yml
$ kubectl config set-context $(kubectl config current-context) --namespace=hands-on
```

<br/>

```
$ for f in kubernetes/helm/components/*; do helm dep up $f; done
$ for f in kubernetes/helm/environments/*; do helm dep up $f; done
```

<br/>

```
$ helm install hands-on-dev-env \
  kubernetes/helm/environments/dev-env \
  -n hands-on --wait
```

<br/>

Possible, need to kill product-composite to start.


<br/>

```
// Should be run already
$ minikube tunnel
```

<br/>

```
$ ./test-em-all.bash
```


<br/>

### Installing a local mail server for tests

<br/>

```
$ kubectl -n istio-system create deployment mail-server --image maildev/maildev:1.1.0
$ kubectl -n istio-system expose deployment mail-server --port=80,25 --type=ClusterIP
$ kubectl -n istio-system wait --timeout=60s --for=condition=ready pod -l app=mail-server
```

<br/>

```
$ helm upgrade istio-hands-on-addons kubernetes/helm/environments/istio-system -n istio-system
```

<br/>

https://mail.minikube.me



```
$ kubectl -n istio-system set env deployment/grafana \
  GF_SMTP_ENABLED=true \
  GF_SMTP_SKIP_VERIFY=true \
  GF_SMTP_HOST=mail-server:25 \
  GF_SMTP_FROM_ADDRESS=grafana@minikube.me

$ kubectl -n istio-system wait --timeout=60s --for=condition=ready pod -l app=grafana
```


<br/>

### Starting up the load test

<br/>

```
$ ACCESS_TOKEN=$(curl -k https://writer:secret@minikube.me/oauth2/token -d grant_type=client_credentials -s | jq .access_token -r)

$ echo ACCESS_TOKEN=${ACCESS_TOKEN}

$ siege https://minikube.me/product-composite/1 -H "Authorization: Bearer ${ACCESS_TOKEN}" -c1 -d1 -v
```

<br/>

### Using Kiali's built-in dashboards

<br/>

// admin / admin  
https://kiali.minikube.me

<br/>

Workloads

<br/>

![Application](/img/ch20-pic01.png?raw=true)


<br/>


https://grafana.minikube.me

<br/>

![Application](/img/ch20-pic02.png?raw=true)

<br/><br/>

---

<br/>

**Marley**

Any questions in english: <a href="https://javadev.org/chat/">Telegram Chat</a>  
Любые вопросы на русском: <a href="https://javadev.ru/chat/">Телеграм чат</a>