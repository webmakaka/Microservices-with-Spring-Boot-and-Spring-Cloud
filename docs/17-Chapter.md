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

### Prepare

<br/>

```
$ cd apps/Chapter17/
```

<br/>

// auth-server

```
$ cd kubernetes/helm/components/auth-server/config-repo/
```

<br/>


```
$ {
  ln -s ../../../../../config-repo/application.yml  application.yml
  ln -s ../../../../../config-repo/auth-server.yml auth-server.yml
}
```


<br/>

// product-composite

```
$ cd kubernetes/helm/components/product-composite/config-repo/
```

<br/>


```
$ {
  ln -s ../../../../../config-repo/application.yml  application.yml
  ln -s ../../../../../config-repo/product-composite.yml product-composite.yml
}
```


<br/>

// product

```
$ cd kubernetes/helm/components/product/config-repo/
```

<br/>


```
$ {
  ln -s ../../../../../config-repo/application.yml  application.yml
  ln -s ../../../../../config-repo/product.yml product.yml
}
```


<br/>

// recommendation

```
$ cd kubernetes/helm/components/recommendation/config-repo/
```

<br/>


```
$ {
  ln -s ../../../../../config-repo/application.yml  application.yml
  ln -s ../../../../../config-repo/recommendation.yml recommendation.yml
}
```

<br/>

// review

```
$ cd kubernetes/helm/components/review/config-repo/
```

<br/>


```
$ {
  ln -s ../../../../../config-repo/application.yml  application.yml
  ln -s ../../../../../config-repo/review.yml review.yml
}
```



<br/>

```
$ cd ../../../../../

$ ls -l kubernetes/helm/components/product/config-repo
```

<br/>

```
lrwxrwxrwx 1 marley marley 42 Jun 13 19:26 application.yml -> ../../../../../config-repo/application.yml
lrwxrwxrwx 1 marley marley 38 Jun 13 19:26 product.yml -> ../../../../../config-repo/product.yml
```

<br/>

```
$ ls kubernetes/helm/components/product/config-repo
application.yml  product.yml
```

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
NAME                                       READY   STATUS    RESTARTS   AGE
cert-manager-7998c69865-v6jrx              1/1     Running   0          43s
cert-manager-cainjector-7b744d56fb-ptqt5   1/1     Running   0          43s
cert-manager-webhook-7d6d4c78bc-6cztp      1/1     Running   0          43s
```

<br/>

```
// Map minikube.me to the IP address we can use to reach the Minikube instance
$ sudo bash -c "echo $(minikube ip) minikube.me | tee -a /etc/hosts"
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
// Resolving Helm chart dependencies
$ for f in kubernetes/helm/components/*; do helm dep up $f; done
$ for f in kubernetes/helm/environments/*; do helm dep up $f; done
$ helm dep ls kubernetes/helm/environments/prod-env/
```

<br/>

```
NAME             	VERSION	REPOSITORY                               	STATUS
common           	1.0.0  	file://../../common                      	ok    
auth-server      	1.0.0  	file://../../components/auth-server      	ok    
product          	1.0.0  	file://../../components/product          	ok    
recommendation   	1.0.0  	file://../../components/recommendation   	ok    
review           	1.0.0  	file://../../components/review           	ok    
product-composite	1.0.0  	file://../../components/product-composite	ok    
zipkin-server    	1.0.0  	file://../../components/zipkin-server    	ok
```

<br/>

```
$ kubectl config set-context $(kubectl config current-context) --namespace=hands-on
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
// Additional terminal
$ kubectl get certificates -w --output-watch-events
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
EVENT      NAME              READY   SECRET            AGE
ADDED      tls-certificate           tls-certificate   0s
MODIFIED   tls-certificate   False   tls-certificate   0s
MODIFIED   tls-certificate   False   tls-certificate   1s
MODIFIED   tls-certificate   False   tls-certificate   1s
MODIFIED   tls-certificate   True    tls-certificate   1s
MODIFIED   tls-certificate   True    tls-certificate   1s
MODIFIED   tls-certificate   True    tls-certificate   1s
```

<br/>

```
$ kubectl get pods
```

<br/>

```
$ kubectl get pods
NAME                                 READY   STATUS    RESTARTS   AGE
auth-server-6b7cf86b66-f896w         1/1     Running   0          74s
product-7bb8dd5dbb-2ctsn             1/1     Running   0          74s
product-composite-85fc98c8bf-qjghd   1/1     Running   1          74s
recommendation-7c9bbd76f8-xrgsx      1/1     Running   0          74s
review-7b48469995-zq6fv              1/1     Running   0          74s
zipkin-server-6b9b4d988f-g9tzg       1/1     Running   0          74s
```

<br/>

```
// logs
$ kubectl logs product-7bb8dd5dbb-2ctsn
```

<br/>

```
$ kubectl get cm 
NAME                DATA   AGE
auth-server         0      33m
kube-root-ca.crt    1      33m
product             0      33m
product-composite   0      33m
recommendation      0      33m
review              0      33m
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


<br/><br/>

---

<br/>

**Marley**

Any questions in english: <a href="https://javadev.org/chat/">Telegram Chat</a>  
Любые вопросы на русском: <a href="https://javadev.ru/chat/">Телеграм чат</a>