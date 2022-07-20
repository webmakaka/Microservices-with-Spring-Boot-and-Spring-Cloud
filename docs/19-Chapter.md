# [Book] [Magnus Larsson] Microservices with Spring Boot and Spring Cloud [ENG, 2021]

<br/>

## Chapter 19. Centralized Logging with the EFK Stack

<br/>

https://github.com/fluent/fluentd-kubernetes-daemonset/tree/master/archived-image/v1.4/debian-elasticsearch/conf

<br/>

https://fluentular.herokuapp.com/

<br/>

### Building and deploying our microservices


**Need to repeat steps from previous chapter**

<br/>

```
$ cd apps/Chapter19
$ eval $(minikube docker-env)
$ ./gradlew build && docker-compose build
```

<br/>

<!--
$ kubectl delete namespace hands-on
-->

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

```
$ minikube tunnel
```

<br/>

```
$ ./test-em-all.bash
```

<br/>

```
$ ACCESS_TOKEN=$(curl -k https://writer:secret@minikube.me/oauth2/token -d grant_type=client_credentials -s | jq .access_token -r)

$ echo ACCESS_TOKEN=${ACCESS_TOKEN}

$ curl -ks https://minikube.me/product-composite/1 -H "Authorization: Bearer ${ACCESS_TOKEN}" | jq .productId
```

<br/>

### Deploying Elasticsearch and Kibana

<br/>

```
$ eval $(minikube docker-env)
// $ docker pull docker.elastic.co/elasticsearch/elasticsearch:7.12.1
$ docker pull elasticsearch:7.12.1
// $ docker pull docker.elastic.co/kibana/kibana:7.12.1
$ docker pull kibana:7.12.1
```

<br/>

```
$ helm install logging-hands-on-add-on kubernetes/helm/environments/logging \
  -n logging --create-namespace --wait
```

<br/>

```
$ kubectl get pods -n logging
NAME                             READY   STATUS    RESTARTS   AGE
elasticsearch-86d8cc5c69-ddvmm   1/1     Running   0          51s
kibana-54ccdcd65-ldjz9           1/1     Running   0          51s
```

<br/>

```
$ curl https://elasticsearch.minikube.me -sk | jq -r .tagline
```

<br/>

Need to wait

<br/>

**response:**

```
You Know, for Search
```

<br/>

```
$ curl https://kibana.minikube.me \
-kLs -o /dev/null -w "%{http_code}\n"
```

<br/>

### Deploying Fluentd

<br/>

```
$ eval $(minikube docker-env)
$ docker build -f kubernetes/efk/Dockerfile -t hands-on/fluentd:v1 kubernetes/efk/
```

<br/>

```
$ kubectl apply -f kubernetes/efk/fluentd-hands-on-configmap.yml
$ kubectl apply -f kubernetes/efk/fluentd-ds.yml
$ kubectl wait --timeout=120s --for=condition=Ready pod -l app=fluentd -n kube-system
```

<br/>

```
$ kubectl logs -n kube-system -l app=fluentd --tail=-1 | grep "fluentd worker is now running worker"
```

<br/>

```
$ curl https://elasticsearch.minikube.me/_all/_count -sk | jq .count
```

<br/>

### Initializing Kibana


https://kibana.minikube.me

<br/>

Explore on my own --> "hamburger menu" --> Visualize Library --> Create index pattern

<br/>

logstash-* --> Next --> @timestamp --> Create index pattern


<br/>

Visualize Library --> Create new visualization --> Lens --> logstash-* --> Pie

<br/><br/>

---

<br/>

**Marley**

Any questions in english: <a href="https://javadev.org/chat/">Telegram Chat</a>  
Любые вопросы на русском: <a href="https://javadev.ru/chat/">Телеграм чат</a>