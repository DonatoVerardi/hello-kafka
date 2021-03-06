# hello-kafka

This is my lab repo for my experiments with kafka on containers.

The goal of this lab is to be able to deploy kafka and the all ecosystem with a simple click.

-> ArgoCD
-> Strimzi
-> Schema Registry
-> Kafka Connect
-> ksqlBD
-> Cruise Control

-> Keycloak
-> Istio / envoy / Kiali
-> Prometheus / Grafana

-> Open Policy Agent / Gateway

Cheers!

## Argocd

> kubectl create ns argocd

> kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

> kubectl expose service argocd-server --port=80 --target-port=8080 --name=argocd --type=NodePort -n argocd

> export INGRESS_HOST=$(minikube ip)

> export ARGOCD_PORT=$(kubectl -n argocd get service argocd -o jsonpath='{.spec.ports[0].nodePort}')

> export ARGOCD_INIT_PWD=$(kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d)

> argocd login $INGRESS_HOST:$ARGOCD_PORT --insecure --username admin --password $ARGOCD_INIT_PWD

> argocd account update-password --current-password $ARGOCD_INIT_PWD --new-password admin1234

> kustomize build ./argocd-root-kustomize/\_bootstrap/ | kubectl apply -f -

> minikube service argocd -n argocd

login with admin / admin1234

## Strimzi

> wget https://github.com/strimzi/strimzi-kafka-operator/releases/download/0.27.1/strimzi-0.27.1.zip -O ./strimzi-kustomize/\_source/strimzi.zip

> unzip ./strimzi-kustomize/\_source/strimzi.zip -d ./strimzi-kustomize/\_source

> sed -i 's/namespace: .*/namespace: strimzi/' ./strimzi-kustomize/\_source/strimzi-0.27.1/install/cluster-operator/*RoleBinding\*.yaml

> Edit the install/cluster-operator/060-Deployment-strimzi-cluster-operator.yaml file and set the STRIMZI_NAMESPACE environment variable to the namespace kafka.

> kubectl create -f ./strimzi-kustomize/\_source/strimzi-0.27.1/install/cluster-operator/ -n strimzi

> kubectl create -f ./strimzi-kustomize/\_source/strimzi-0.27.1/install/cluster-operator/020-RoleBinding-strimzi-cluster-operator.yaml -n kafka

## istio

> istioctl install --set profile=demo -y

> kubectl label namespace kafka istio-injection=enabled

> istioctl dashboard kiali

> minikube tunnel

# test


> kubectl -n kafka run kafka-producer -ti --image=quay.io/strimzi/kafka:0.27.1-kafka-3.0.0 --rm=true --restart=Never -- bin/kafka-console-producer.sh --broker-list my-cluster-kafka-bootstrap:9092 --topic my-topic

> kubectl -n kafka run kafka-consumer -ti --image=quay.io/strimzi/kafka:0.27.1-kafka-3.0.0 --rm=true --restart=Never -- bin/kafka-console-consumer.sh --bootstrap-server my-cluster-kafka-bootstrap:9092 --topic my-topic --from-beginning