# [Book] [Magnus Larsson] Microservices with Spring Boot and Spring Cloud [ENG, 2021]

<br/>

## Chapter 17. Implementing Kubernetes Features to Simplify the System Landscape

<br/>

## Kubernetes

<br/>

**The following topics will be covered in this chapter:**

• Replacing the Spring Cloud Config Server with Kubernetes ConfigMaps and Secrets
• Replacing the Spring Cloud Gateway with a Kubernetes Ingress object
• Using the cert-manager to automatically provision certificates
• Deploying and testing the microservice landscape on Kubernetes
• Deploying and testing the microservice landscape using Docker Compose to ensure that the source code in the microservices isn't locked into Kubernetes

<br/>

![Application](/img/ch17-pic01.png?raw=true)

<br/>

![Application](/img/ch17-pic02.png?raw=true)


<br/>

Since self-signed certificates don't require communication with any external resources, they are a good candidate for use during development. 


<br/>

### [Run Minikube](15-Chapter.md)

<br/>

### Install the cert-manager

<br/>

```
$ helm repo add jetstack https://charts.jetstack.io

$ helm repo update

$ helm install cert-manager jetstack/cert-manager \
--create-namespace \
--namespace cert-manager \
--version v1.3.1 \
--set installCRDs=true \
--wait
```

<br/>

```
$ kubectl get pods --namespace cert-manager
```

<br/>

```
// Map minikube.me to the IP address we can use to reach the Minikube instance
$ sudo bash -c "echo $(minikube ip) minikube.me | tee -a /etc/hosts"
```


<br/>

```
// Resolving Helm chart dependencies
$ for f in kubernetes/helm/components/*; do helm dep up $f; done
$ for f in kubernetes/helm/environments/*; do helm dep up $f; done
$ helm dep ls kubernetes/helm/environments/prod-env/
```

<br/>

```
$ kubectl config set-context $(kubectl config current-context) --namespace=hands-on
```


<br/>

## Deploying to Kubernetes for staging and production

<br/>

```
$ eval $(minikube docker-env)
$ ./gradlew build && docker-compose build
```

<br/>

```
$ docker-compose up -d mongodb mysql rabbitmq
```

<br/>

```
$ {
  docker tag hands-on/auth-server hands-on/auth-server:v1
  docker tag hands-on/product-composite-service hands-on/product-composite-service:v1
  docker tag hands-on/product-service hands-on/product-service:v1
  docker tag hands-on/recommendation-service hands-on/recommendation-service:v1
  docker tag hands-on/review-service hands-on/review-service:v1
}
```

<br/>

```
$ helm install hands-on-prod-env \
kubernetes/helm/environments/prod-env \
-n hands-on --create-namespace \
--wait
```

<br/>

```
$ kubectl get pods -n hands-on
```

<br/>

```
$ HOST=minikube.me PORT=443 USE_K8S=true ./test-em-all.bash
```

<br/>

```
$ kubectl get cert -w
$ kubectl get events -w
```

<br/>

```
// Delete
$ kubectl delete namespace hands-on
```

<br/>
<hr/>
<br/>

### [Additional] Working with certificates

<br/>

```
$ kubectl get certificates -w --output-watch-events
```

<br/>

```
$ helm install hands-on-dev-env \
kubernetes/helm/environments/dev-env \
-n hands-on \
--create-namespace \
--wait
```

<br/>

```
$ kubectl get certificates
```

<br/>

```
$ HOST=minikube.me PORT=443 USE_K8S=true ./test-em-all.bash
```

<br/>

### Rotating certificates

<br/>

```
$ kubectl describe cert tls-certificate
```

<br/>

```
$ kubectl patch certificate tls-certificate --type=json \
-p='[{"op": "add", "path": "/spec/renewBefore", "value": "2159h59m"}]'
```

<br/>

```
$ kubectl get events -w
```

<br/>

```
$ kubectl get cert tls-certificate -o json | jq .status.renewalTime
```

<br/>

```
$ kubectl patch certificate tls-certificate --type=json \
-p='[{"op": "remove", "path": "/spec/renewBefore"}]'
```

<br/>

```
$ kubectl delete namespace hands-on
```

<br/>

## Docker (without Kubernetes)

<br/>

```
$ COMPOSE_FILE=docker-compose.yml ./test-em-all.bash start stop
```

<br/>

```
$ COMPOSE_FILE=docker-compose-partitions.yml ./test-em-all.bash start stop
```

<br/>

```
$ COMPOSE_FILE=docker-compose-kafka.yml ./test-em-all.bash start stop
```

<br/>
<hr/>
<br/>


<br/><br/>

---

<br/>

**Marley**

Any questions in english: <a href="https://javadev.org/chat/">Telegram Chat</a>  
Любые вопросы на русском: <a href="https://javadev.ru/chat/">Телеграм чат</a>