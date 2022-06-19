# Avalanche-Node
Deployment for an [Avalanche](https://github.com/ava-labs/avalanchego) node using Kubernetes.

For the purpose of this exercise, following tools will be used
- docker
- [minikube](https://minikube.sigs.k8s.io/docs/start/)
- golang (>=1.17.9 except v1.18.x)
- [helm3](https://helm.sh/docs/intro/install/#from-the-binary-releases)

## Startup minikube
For this exercise, minikube will be used to run kubernetes cluster locally on ubuntu 20.04 LTS

`minikube start --cpus 8 --memory 10g`

## Build Avalanche Node docker image
Using the scripts provided from Avalanchego github to build docker image from source https://github.com/ava-labs/avalanchego#docker-install

```
$ git clone https://github.com/ava-labs/avalanchego.git

$ ./scripts/build_image.sh

$ docker image ls 
```
Am image with name and tag `avalanchego:master` will be created

## Deploy Avalanche Node
Upload docker image builded from source to minikube 

`minikube image load avalanchego:master`

Deployment k8s resources - deployment, PV, PVC and service

`kubectl apply -f .`


## Install Grafana/Prometheus onto k8s
To monitor this instances of Avalanche node, we use Grafana/Premetheus to gather metrics and visualize data. 

###  First, we'll need to install Prometheus Operator via the community helm chart  

```
$ helm repo add prometheus-community https://prometheus-community.github.io/helm-charts

$ helm repo update

$ helm install monitoring prometheus-community/kube-prometheus-stack
```

### Login into Grafana

```
# expose grafana UI http endpoint 
$ kubectl port-forward svc/monitoring-grafana 8080:80

# Default Username is admin
# Get password 
$ kubectl get secret  monitoring-grafana -o json | jq -r .'["data"]["admin-password"]' | base64 --decode
```

Once logged into Grafana UI, we can upload and use [pre-made Dashboards](https://github.com/ava-labs/avalanche-monitoring/tree/main/grafana/dashboards)  

Source Reference: https://docs.avax.network/nodes/maintain/setting-up-node-monitoring
