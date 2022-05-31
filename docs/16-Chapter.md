# [Book] [Magnus Larsson] Microservices with Spring Boot and Spring Cloud [ENG, 2021]

<br/>

## Chapter 16. Deploying Our Microservices in Kubernetes



<br/>

![Application](/img/ch16-pic01.png?raw=true)

<br/>

### Deploying to Kubernetes for development and test

<br/>

```
$ cd apps/Chapter16/

$ cd kubernetes/helm/components/config-server

$ ln -s ../../../../config-repo config-repo

$ ls -l kubernetes/helm/components/config-server/config-repo

$ ls kubernetes/helm/components/config-server/config-repo
application.yml  gateway.yml            product.yml         review.yml
auth-server.yml  product-composite.yml  recommendation.yml
```

<br/>

```
$ cd apps/Chapter16/

// directs the local Docker client to communicate with the Docker engine in Minikube
$ eval $(minikube docker-env)
$ ./gradlew build && docker-compose build
```

<br/>

### Resolving Helm chart dependencies

```
$ for f in kubernetes/helm/components/*; do helm dep up $f; done
$ for f in kubernetes/helm/environments/*; do helm dep up $f; done
$ helm dep ls kubernetes/helm/environments/dev-env/
```

<br/>

```
NAME             	VERSION	REPOSITORY                               	STATUS
common           	1.0.0  	file://../../common                      	ok    
rabbitmq         	1.0.0  	file://../../components/rabbitmq         	ok    
mongodb          	1.0.0  	file://../../components/mongodb          	ok    
mysql            	1.0.0  	file://../../components/mysql            	ok    
config-server    	1.0.0  	file://../../components/config-server    	ok    
gateway          	1.0.0  	file://../../components/gateway          	ok    
auth-server      	1.0.0  	file://../../components/auth-server      	ok    
product          	1.0.0  	file://../../components/product          	ok    
recommendation   	1.0.0  	file://../../components/recommendation   	ok    
review           	1.0.0  	file://../../components/review           	ok    
product-composite	1.0.0  	file://../../components/product-composite	ok    
zipkin-server    	1.0.0  	file://../../components/zipkin-server    	ok    
```


<br/>

### Deploying to Kubernetes


<br/>

```
// Check
// render the templates using the helm template command to see what the manifests will look like
$ helm template kubernetes/helm/environments/dev-env
```

<br/>

```
// Check
$ helm install --dry-run --debug hands-on-dev-env \
kubernetes/helm/environments/dev-env
```

<br/>

```
// Create
$ helm install hands-on-dev-env \
kubernetes/helm/environments/dev-env \
-n hands-on \
--create-namespace
```

<br/>

```
$ kubectl config set-context $(kubectl config current-context) --namespace=hands-on
```

<br/>

```
$ kubectl get pods --watch
```

<br/>

```
$ kubectl exec deploy/config-server -- curl dev-usr:dev-pwd@localhost:8888/product/docker -s | jq .
```

**response:**

```
{
  "name": "product",
  "profiles": [
    "docker"
  ],
  "label": null,
  "version": null,
  "state": null,
  "propertySources": [
    {
      "name": "Config resource 'file [/config-repo/product.yml]' via location 'file:/config-repo/' (document #1)",
      "source": {
        "spring.config.activate.on-profile": "docker",
        "server.port": 80,
        "spring.data.mongodb.host": "mongodb"
      }
    },
    {
      "name": "Config resource 'file [/config-repo/product.yml]' via location 'file:/config-repo/' (document #0)",
      "source": {
        "server.port": 7001,
        "server.error.include-message": "always",
        "spring.data.mongodb.host": "localhost",
        "spring.data.mongodb.port": 27017,
        "spring.data.mongodb.database": "product-db",
        "spring.cloud.function.definition": "messageProcessor",
        "spring.cloud.stream.default.contentType": "application/json",
        "spring.cloud.stream.bindings.messageProcessor-in-0.destination": "products",
        "spring.cloud.stream.bindings.messageProcessor-in-0.group": "productsGroup",
        "spring.cloud.stream.bindings.messageProcessor-in-0.consumer.maxAttempts": 3,
        "spring.cloud.stream.bindings.messageProcessor-in-0.consumer.backOffInitialInterval": 500,
        "spring.cloud.stream.bindings.messageProcessor-in-0.consumer.backOffMaxInterval": 1000,
        "spring.cloud.stream.bindings.messageProcessor-in-0.consumer.backOffMultiplier": 2,
        "spring.cloud.stream.rabbit.bindings.messageProcessor-in-0.consumer.autoBindDlq": true,
        "spring.cloud.stream.rabbit.bindings.messageProcessor-in-0.consumer.republishToDlq": true,
        "spring.cloud.stream.kafka.bindings.messageProcessor-in-0.consumer.enableDlq": true,
        "logging.level.root": "INFO",
        "logging.level.se.magnus": "DEBUG",
        "logging.level.org.springframework.data.mongodb.core.ReactiveMongoTemplate": "DEBUG"
      }
    },
    {
      "name": "Config resource 'file [/config-repo/application.yml]' via location 'file:/config-repo/' (document #1)",
      "source": {
        "spring.config.activate.on-profile": "docker",
        "spring.rabbitmq.host": "rabbitmq",
        "spring.cloud.stream.kafka.binder.brokers": "kafka",
        "app.auth-server": "auth-server"
      }
    },
    {
      "name": "Config resource 'file [/config-repo/application.yml]' via location 'file:/config-repo/' (document #0)",
      "source": {
        "app.auth-server": "localhost",
        "spring.rabbitmq.host": "127.0.0.1",
        "spring.rabbitmq.port": 5672,
        "spring.rabbitmq.username": "guest",
        "spring.cloud.stream.kafka.binder.brokers": "127.0.0.1",
        "spring.cloud.stream.kafka.binder.defaultBrokerPort": 9092,
        "spring.cloud.stream.defaultBinder": "rabbit",
        "spring.zipkin.sender.type": "rabbit",
        "spring.sleuth.sampler.probability": 1,
        "management.endpoint.health.show-details": "ALWAYS",
        "management.endpoints.web.exposure.include": "*",
        "management.endpoint.health.probes.enabled": true,
        "management.endpoint.health.group.readiness.include": "rabbit, db, mongo",
        "server.shutdown": "graceful",
        "spring.lifecycle.timeout-per-shutdown-phase": "10s",
        "spring.rabbitmq.password": "guest"
      }
    }
  ]
}
```

<br/>

```
$ kubectl -n hands-on exec deploy/config-server -- ls /config-repo
```

<br/>

**response:**

```
pplication.yml
auth-server.yml
gateway.yml
product-composite.yml
product.yml
recommendation.yml
review.yml
```

<br/>

```
$ kubectl -n hands-on get cm config-server -o yaml
```

<br/>

```
$ kubectl wait --timeout=600s --for=condition=ready pod --all
```

<br/>

```
$ kubectl get pods 
NAME                                 READY   STATUS    RESTARTS   AGE
auth-server-75bdb6949c-5rrdp         1/1     Running   0          2m8s
config-server-5bddc4c765-5dnfx       1/1     Running   0          2m8s
gateway-768df96dcb-bhfms             1/1     Running   1          2m8s
mongodb-bccb55966-bvrzw              1/1     Running   0          2m8s
mysql-665d74c488-z5hq6               1/1     Running   0          2m8s
product-7594bd9bb6-q6kfj             1/1     Running   0          2m7s
product-composite-58744db6d4-znt9z   1/1     Running   1          2m8s
rabbitmq-6dc45b74b-97cv9             1/1     Running   0          2m8s
recommendation-798789d9cb-b5ktl      1/1     Running   0          2m8s
review-667d8f6dd5-qp52q              1/1     Running   0          2m7s
zipkin-server-7b9fcd4cfd-rmmxg       1/1     Running   2          2m7s
```

<br/>

```
$ kubectl get pods -o json | jq .items[].spec.containers[].image
```

<br/>

**response:**

```
"hands-on/auth-server:latest"
"hands-on/config-server:latest"
"hands-on/gateway:latest"
"registry.hub.docker.com/library/mongo:4.4.2"
"registry.hub.docker.com/library/mysql:5.7.32"
"hands-on/product-service:latest"
"hands-on/product-composite-service:latest"
"registry.hub.docker.com/library/rabbitmq:3.8.11-management"
"hands-on/recommendation-service:latest"
"hands-on/review-service:latest"
"registry.hub.docker.com/openzipkin/zipkin:2.23.2"
```

<br/>

### Testing the deployment

```
$ MINIKUBE_HOST=$(minikube ip)
```

<br/>

```
// OK!
$ HOST=$MINIKUBE_HOST PORT=30443 USE_K8S=true ./test-em-all.bash
```


<br/>

### [Skipped] Testing Spring Boot's support for graceful shutdown and probes for liveness and readiness


```
$ ACCESS_TOKEN=$(curl -d grant_type=client_credentials \
-ks https://writer:secret@$MINIKUBE_HOST:30443/oauth2/token \
| jq .access_token -r)
```

<br/>

```
$ echo $ACCESS_TOKEN
```

<br/>

```
$ time curl -kH "Authorization: Bearer $ACCESS_TOKEN" \
https://$MINIKUBE_HOST:30443/product-composite/1?delay=5
```

<br/>

```
$ sudo apt install -y siege
$ siege -c5 -d2 -v -H "Authorization: Bearer $ACCESS_TOKEN" \
https://$MINIKUBE_HOST:30443/product-composite/1?delay=5
```

<br/>

```
$ kubectl logs -f --tail=0 -l app.kubernetes.io/name=product
```

<br/>

```
$ siege -c5 -d5 -v -H "Authorization: Bearer $ACCESS_TOKEN" \
https://$MINIKUBE_HOST:30443/product-composite/1?delay=15
```

<br/>

```
$ kubectl logs -f --tail=0 -l app.kubernetes.io/name=product
```

<br/>

```
$ kubectl exec -it deploy/product -- \
curl localhost/actuator/health/liveness -s | jq .
```

<br/>

```
{
  "status": "UP"
}
```

<br/>

```
$ kubectl exec -it deploy/product -- \
curl localhost/actuator/health/readiness -s | jq .
```

<br/>


```
{
  "status": "UP",
  "components": {
    "mongo": {
      "status": "UP",
      "details": {
        "version": "4.4.2"
      }
    },
    "rabbit": {
      "status": "UP",
      "details": {
        "version": "3.8.11"
      }
    }
  }
}
```

<br/>

```
// Delete the Namespace
$ kubectl delete namespace hands-on

or

$ helm uninstall hands-on-dev-env

```

<br/>

### Deploying to Kubernetes for staging and production

<br/>

```
$ eval $(minikube docker-env)
$ docker-compose up -d mongodb mysql rabbitmq
```

<br/>

```
$ {
  docker tag hands-on/auth-server hands-on/auth-server:v1
  docker tag hands-on/config-server hands-on/config-server:v1
  docker tag hands-on/gateway hands-on/gateway:v1
  docker tag hands-on/product-composite-service hands-on/product-composite-service:v1
  docker tag hands-on/product-service hands-on/product-service:v1
  docker tag hands-on/recommendation-service hands-on/recommendation-service:v1
  docker tag hands-on/review-service hands-on/review-service:v1
}
```

<br/>

```
$ helm install hands-on-prod-env \ kubernetes/helm/environments/prod-env \
-n hands-on --create-namespace
```


<br/>

```
$ kubectl get pods --watch
```

<br/>

```
$ kubectl wait --timeout=600s --for=condition=ready pod --all
```

<br/>

```
$ kubectl get pods -o json | jq .items[].spec.containers[].image
```

<br/>

```
"hands-on/auth-server:v1"
"hands-on/config-server:v1"
"hands-on/gateway:v1"
"hands-on/product-service:v1"
"hands-on/product-composite-service:v1"
"hands-on/recommendation-service:v1"
"hands-on/review-service:v1"
"registry.hub.docker.com/openzipkin/zipkin:2.23.2"
```


<br/>

```
$ CONFIG_SERVER_USR=prod-usr \
CONFIG_SERVER_PWD=prod-pwd \
HOST=$MINIKUBE_HOST PORT=30443 USE_K8S=true ./test-em-all.bash
```

<br/>

```
$ kubectl exec deploy/config-server -- curl prod-usr:prod-pwd@localhost:8888/product/docker -s | jq .
```


<br/>

### Cleaning up

```
$ kubectl delete namespace hands-on
```

<br/>

```
$ eval $(minikube docker-env)
$ docker-compose down
```

<br/>
<hr/>
<br/>

## Check Helm configs

<br/>

### The ConfigMap template


```
$ cd apps/Chapter16/kubernetes/helm/components/config-server
$ helm dependency update .
$ helm template . -s templates/configmap_from_file.yaml
```

<br/>

```
---
# Source: config-server/templates/configmap_from_file.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: config-server
  labels:
    app.kubernetes.io/name: config-server
    helm.sh/chart: config-server-1.0.0
    app.kubernetes.io/managed-by: Helm
data:
  {}
```

<br/>

### The Secrets template

```
$ cd apps/Chapter16/kubernetes/helm
$ for f in components/*; do helm dependency update $f; done
$ helm dependency update environments/dev-env
$ helm template environments/dev-env -s templates/secrets.yaml
```

<br/>

```
---
# Source: dev-env/templates/secrets.yaml
apiVersion: v1
kind: Secret
metadata:
  name: config-client-credentials
  labels:
    app.kubernetes.io/name: config-client-credentials
    helm.sh/chart: dev-env-1.0.0
    app.kubernetes.io/managed-by: Helm
type: Opaque
data:
  CONFIG_SERVER_PWD: ZGV2LXB3ZA==
  CONFIG_SERVER_USR: ZGV2LXVzcg==
---
# Source: dev-env/templates/secrets.yaml
apiVersion: v1
kind: Secret
metadata:
  name: config-server-secrets
  labels:
    app.kubernetes.io/name: config-server-secrets
    helm.sh/chart: dev-env-1.0.0
    app.kubernetes.io/managed-by: Helm
type: Opaque
data:
  ENCRYPT_KEY: bXktdmVyeS1zZWN1cmUtZW5jcnlwdC1rZXk=
  SPRING_SECURITY_USER_NAME: ZGV2LXVzcg==
  SPRING_SECURITY_USER_PASSWORD: ZGV2LXB3ZA==
```

<br/>

### The Service template

```
$ cd apps/Chapter16/kubernetes/helm
$ helm dependency update components/product
$ helm template components/product -s templates/service.yaml
```

<br/>

```
---
# Source: product/templates/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: product
  labels:
    app.kubernetes.io/name: product
    helm.sh/chart: product-1.0.0
    app.kubernetes.io/managed-by: Helm
spec:
  type: ClusterIP
  ports:
    - name: http
      port: 80
      protocol: TCP
      targetPort: http
  selector:
    app.kubernetes.io/name: product
```

<br/>


```
$ helm dependency update components/gateway
$ helm template components/gateway -s templates/service.yaml
```

<br/>

```
---
# Source: gateway/templates/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: gateway
  labels:
    app.kubernetes.io/name: gateway
    helm.sh/chart: gateway-1.0.0
    app.kubernetes.io/managed-by: Helm
spec:
  type: NodePort
  ports:
    - nodePort: 30443
      port: 443
      targetPort: 8443
  selector:
    app.kubernetes.io/name: gateway
```

<br/>

### The Deployment template

<br/>


```
$ helm dependency update components/product
$ helm template components/product -s templates/deployment.yaml
```

<br/>

```
---
# Source: product/templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: product
  labels:
    app.kubernetes.io/name: product
    helm.sh/chart: product-1.0.0
    app.kubernetes.io/managed-by: Helm
spec:
  replicas: 1
  selector:
    matchLabels:
      app.kubernetes.io/name: product
  template:
    metadata:
      labels:
        app.kubernetes.io/name: product
    spec:
      containers:
        - name: product
          image: "hands-on/product-service:latest"
          imagePullPolicy: Never
          env:
          - name: SPRING_PROFILES_ACTIVE
            value: docker
          livenessProbe:
            failureThreshold: 20
            httpGet:
              path: /actuator/health/liveness
              port: 80
              scheme: HTTP
            initialDelaySeconds: 10
            periodSeconds: 10
            successThreshold: 1
            timeoutSeconds: 2
          readinessProbe:
            failureThreshold: 3
            httpGet:
              path: /actuator/health/readiness
              port: 80
              scheme: HTTP
            initialDelaySeconds: 10
            periodSeconds: 10
            successThreshold: 1
            timeoutSeconds: 2
          ports:
            - containerPort: 80
              name: http
              protocol: TCP
          resources:
            limits:
              memory: 350Mi
```

<br/>

```
$ helm dependency update components/mongodb
$ helm template components/mongodb -s templates/deployment.yaml
```

<br/>

```
---
# Source: mongodb/templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongodb
  labels:
    app.kubernetes.io/name: mongodb
    helm.sh/chart: mongodb-1.0.0
    app.kubernetes.io/managed-by: Helm
spec:
  replicas: 1
  selector:
    matchLabels:
      app.kubernetes.io/name: mongodb
  template:
    metadata:
      labels:
        app.kubernetes.io/name: mongodb
    spec:
      containers:
        - name: mongodb
          image: "registry.hub.docker.com/library/mongo:4.4.2"
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: 27017
          resources:
            limits:
              memory: 350Mi
```


<br/><br/>

---

<br/>

**Marley**

Any questions in english: <a href="https://javadev.org/chat/">Telegram Chat</a>  
Любые вопросы на русском: <a href="https://javadev.ru/chat/">Телеграм чат</a>