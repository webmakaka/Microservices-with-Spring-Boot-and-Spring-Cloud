# [Book] [Magnus Larsson] Microservices with Spring Boot and Spring Cloud [ENG, 2021]

<br/>

## Chapter 18. Using a Service Mesh to Improve Observability and Management


<br/>

```
$ minikube status
handson-spring-boot-cloud
type: Control Plane
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured
docker-env: in-use
```

<br/>

### Setup istioctl

```
$ cd ~/tmp/
$ export LATEST_VERSION=$(curl --silent "https://api.github.com/repos/istio/istio/releases/latest" | grep '"tag_name"' | sed -E 's/.*"([^"]+)".*/\1/')

$ curl -L https://istio.io/downloadIstio | sh - && chmod +x ./istio-${LATEST_VERSION}/bin/istioctl && sudo mv ./istio-${LATEST_VERSION}/bin/istioctl /usr/local/bin/
```

<br/>

```
$ istioctl experimental precheck
✔ No issues found when checking the cluster. Istio is safe to install or upgrade!
```

<br/>

```
// Install Istio using the demo profile
$ istioctl install --skip-confirmation \
  --set profile=demo \
  --set meshConfig.accessLogFile=/dev/stdout \
  --set meshConfig.accessLogEncoding=JSON
```

<br/>

```
$ kubectl -n istio-system wait --timeout=600s --for=condition=available deployment --all
```

<br/>

```
// install Kiali, Jaeger, Prometheus, and Grafana
$ istio_version=$(istioctl version --short --remote=false) 

$ echo "Installing integrations for Istio v$istio_version"

$ {
  kubectl apply -n istio-system -f https://raw.githubusercontent.com/istio/istio/${istio_version}/samples/addons/kiali.yaml

  kubectl apply -n istio-system -f https://raw.githubusercontent.com/istio/istio/${istio_version}/samples/addons/jaeger.yaml

  kubectl apply -n istio-system -f https://raw.githubusercontent.com/istio/istio/${istio_version}/samples/addons/prometheus.yaml

  kubectl apply -n istio-system -f https://raw.githubusercontent.com/istio/istio/${istio_version}/samples/addons/grafana.yaml
}
```

<br/>

```
$ kubectl -n istio-system wait --timeout=600s --for=condition=available deployment --all
```

<br/>


```
$ kubectl -n istio-system get deploy
NAME                   READY   UP-TO-DATE   AVAILABLE   AGE
grafana                1/1     1            1           76s
istio-egressgateway    1/1     1            1           5m7s
istio-ingressgateway   1/1     1            1           5m7s
istiod                 1/1     1            1           5m21s
jaeger                 1/1     1            1           92s
kiali                  1/1     1            1           99s
prometheus             1/1     1            1           83s
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
$ cd apps/Chapter18

// Mandatory step!
$ helm upgrade --install istio-hands-on-addons kubernetes/helm/environments/istio-system -n istio-system --wait
```

<br/>

```
$ kubectl -n istio-system get secret hands-on-certificate
NAME                   TYPE                DATA   AGE
hands-on-certificate   kubernetes.io/tls   3      10s
```

<br/>

```
$ kubectl -n istio-system get certificate hands-on-certificate
NAME                   READY   SECRET                 AGE
hands-on-certificate   True    hands-on-certificate   20s
```

<br/>

```
$ minikube tunnel
```

<!--

<br/>

```
// Map minikube.me to the IP address we can use to reach the Minikube instance
$ sudo bash -c "echo $(minikube ip) minikube.me | tee -a /etc/hosts"
```
-->

<br/>

```
$ INGRESS_IP=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.status.loadBalancer.ingress[0].ip}')

$ echo ${INGRESS_IP}
```

<br/>

```
$ MINIKUBE_HOSTS="minikube.me grafana.minikube.me kiali.minikube.me prometheus.minikube.me tracing.minikube.me kibana.minikube.me elasticsearch.minikube.me mail.minikube.me health.minikube.me"

$ echo "$INGRESS_IP $MINIKUBE_HOSTS" | sudo tee -a /etc/hosts
```

<br/>

```
// 200
$ curl -o /dev/null -sk -L -w "%{http_code}\n" https://kiali.minikube.me/kiali/
$ curl -o /dev/null -sk -L -w "%{http_code}\n" https://tracing.minikube.me
$ curl -o /dev/null -sk -L -w "%{http_code}\n" https://grafana.minikube.me
$ curl -o /dev/null -sk -L -w "%{http_code}\n" https://prometheus.minikube.me/graph#/
```

<br/>

### Running commands to create the service mesh


```
$ cd apps/Chapter18
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
// update the dependencies in the components folder
$ for f in kubernetes/helm/components/*; do helm dep up $f; done

// update the dependencies in the environments folder
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
$ kubectl get pods
```

<br/>

```
NAME                                 READY   STATUS    RESTARTS   AGE
auth-server-57f69694cd-8l2wn         2/2     Running   0          58s
mongodb-65f97497f8-cfm6n             1/1     Running   0          58s
mysql-7fb595587-5w79t                1/1     Running   0          58s
product-6658ff976c-pvctx             2/2     Running   0          58s
product-composite-6ff4c96688-whk9s   2/2     Running   1          58s
rabbitmq-7df5dcdcfc-746tm            1/1     Running   0          58s
recommendation-76ffc999f9-dhj44      2/2     Running   0          58s
review-67499bcfdc-nt6pt              1/2     Running   0          58s
```

<br/>


```
// OK!
$ ./test-em-all.bash
```

<br/>

```
$ ACCESS_TOKEN=$(curl -k https://writer:secret@minikube.me/oauth2/token -d grant_type=client_credentials -s | jq .access_token -r)

$ echo ACCESS_TOKEN=${ACCESS_TOKEN}

$ curl -ks https://minikube.me/product-composite/1 -H "Authorization: Bearer ${ACCESS_TOKEN}" | jq .productId
```

<br/>


```
$ siege https://minikube.me/product-composite/1 -H "Authorization: Bearer ${ACCESS_TOKEN}" -c1 -d1 -v
```


<br/>


https://kiali.minikube.me


Overview -> hands-on -> Graph


<br/>

![Application](/img/ch18-pic01.png?raw=true)



<br/>

https://tracing.minikube.me

<br/>

![Application](/img/ch18-pic02.png?raw=true)

<br/>

![Application](/img/ch18-pic03.png?raw=true)


<br/>

### Securing a service mesh


<br/>

```
$ keytool -printcert -sslserver minikube.me | grep -E "Owner:|Issuer:"
Owner: SERIALNUMBER=my-sn, CN=minikube.me, OU=my-ou, O=my-org, OID.2.5.4.17=my-pc, STREET=my-address, L=my-locality, ST=my-province, C=my-country
Issuer: CN=hands-on-ca
```

<br/>

```
$ kubectl apply -f kubernetes/resilience-tests/product-virtual-service-with-faults.yml
```

<br/>

```
$ kubectl delete -f kubernetes/resilience-tests/product-virtual-service-with-faults.yml
```

<br/>

```
$ kubectl apply -f kubernetes/resilience-tests/product-virtual-service-with-delay.yml
```

<br/>

```
for i in {1..6}; do time curl -k https://minikube.me/product-composite/1 -H "Authorization: Bearer $ACCESS_TOKEN"; done
```

```
$ kubectl delete -f kubernetes/resilience-tests/product-virtual-service-with-delay.yml
```


<br/>

### Deploying v1 and v2 versions of the microservices with routing to the v1 version

<br/>

```
$ helm uninstall hands-on-dev-env
```

<br/>

```
$ kubectl get pods
```

<br/>

```
$ eval $(minikube docker-env)
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

    docker tag hands-on/product-service hands-on/product-service:v2
    docker tag hands-on/recommendation-service hands-on/recommendation-service:v2
    docker tag hands-on/review-service hands-on/review-service:v2
}
```

<br/>

```
$ helm install hands-on-prod-env \
  kubernetes/helm/environments/prod-env \
  -n hands-on --wait
```

<br/>

```
// SOMETHING BAD!
$ kubectl get pods
NAME                                 READY   STATUS             RESTARTS   AGE
auth-server-59db665f86-wmg8p         2/2     Running            0          2m28s
product-composite-6fb6dd6c88-6xpxf   2/2     Running            1          2m28s
product-v1-bcddb465c-9tqq8           2/2     Running            0          2m28s
product-v2-596cd48c44-nw87p          1/2     CrashLoopBackOff   4          2m28s
recommendation-v1-7b88f567d5-kwffb   2/2     Running            0          2m28s
recommendation-v2-55f469877d-hfn8s   1/2     CrashLoopBackOff   4          2m28s
review-v1-54d9fcbddc-lrt5l           2/2     Running            0          2m28s
review-v2-7bd8c5b4bd-6zq5x           1/2     CrashLoopBackOff   4          2m28s
```

<br/>

```
// SHOULD BE FAIL!
$ ./test-em-all.bash
```

<br/>

```
$ kubectl -n istio-system delete pod -l app=istiod
```

<br/>

```
// SHOULD BE OK!
$ ./test-em-all.bash
```

<br/>

### Running canary tests

// TODO


<br/>

### Running blue/green deployment


// TODO

<br/>

### Running tests with Docker Compose

<br/>

**Not in Minikube!**

```
// OK!
$ USE_K8S=false HOST=localhost PORT=8443 HEALTH_URL=https://localhost:8443 ./test-em-all.bash start stop
```

<br/><br/>

---

<br/>

**Marley**

Any questions in english: <a href="https://javadev.org/chat/">Telegram Chat</a>  
Любые вопросы на русском: <a href="https://javadev.ru/chat/">Телеграм чат</a>