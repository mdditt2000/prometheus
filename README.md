# Prometheus Example - BIGIP with Telemetry Streaming and Container Ingress Services
Prometheus is an open source monitoring framework. Explaining Prometheus is out of the scope of this repo. In this repo, I will guide you to setup Prometheus on a BIGIP and use Telemetry Streaming and collect metrics

## How to Setup Prometheus Monitoring On Kubernetes Cluster

### Create a Namespace and ClusterRole
Create a Kubernetes namespace for all our monitoring components
```
kubectl create namespace monitoring
```
Create a file named clusterRole.yaml. Locate the clusterRole.yaml file from my repo [yaml](https://github.com/mdditt2000/prometheus/blob/master/clusterRole.yaml)
```
kubectl create -f clusterRole.yaml
```
### Create a Config Map
We should create a config map with all the prometheus scrape config and alerting rules, which will be mounted to the Prometheus container in /etc/prometheus as prometheus.yaml and prometheus.rules files

Create a file called config-map.yaml. Locate the config-map.yaml file from my repo [yaml](https://github.com/mdditt2000/prometheus/blob/master/config-map.yaml)
```
kubectl create -f config-map.yaml
```
The prometheus.yaml contains all the configuration to dynamically discover pods and services running in the Kubernetes cluster. We have the following scrape jobs in our Prometheus scrape configuration. For more information review the following from devopscube.com [link](https://devopscube.com/setup-prometheus-monitoring-on-kubernetes/)

### Create a Prometheus Deployment
Create a file named prometheus-deployment.yaml. Locate the prometheus-deployment.yaml file from my repo [yaml](https://github.com/mdditt2000/prometheus/blob/master/prometheus-deployment.yaml)
```
kubectl create  -f prometheus-deployment.yaml
```
You can check the created deployment using the following command
```
[kube@k8s-1-18-master prometheus]$ kubectl get deployments --namespace=monitoring
NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
prometheus-deployment   1/1     1            1           10d
[kube@k8s-1-18-master prometheus]$
```
## Connecting To Prometheus Dashboard via F5 Container Ingress Services

You can view the deployed Prometheus dashboard in two ways.

* Using Kubectl port forwarding
* Exposing the Prometheus deployment as a service with F5 Load Balancer using Container Ingress Services

### Using Kubectl port forwarding

Follow the using Kubectl port forwarding steps documented at devopscube.com [link](https://devopscube.com/setup-prometheus-monitoring-on-kubernetes/)

### Exposing the Prometheus deployment as a service with F5 Load Balancer using Container Ingress Services

To access the Prometheus dashboard over a IP or a DNS name, you need to expose it as Kubernetes service.

## Create a file named prometheus-service.yaml
We will expose Prometheus using ClusterIP. ClusterIP allows the BIGIP to forward traffic directly to the Prometheus Pod bypassing kube-proxy. which will create a load balancer and points it to the service. You can use ClusterIP type, which will create a F5 BIGIP load balancer and points it to the service
```
apiVersion: v1
kind: Service
metadata:
  name: prometheus-service
  namespace: monitoring
  annotations:
      prometheus.io/scrape: 'true'
      prometheus.io/port:   '9090'
  
spec:
  selector: 
    app: prometheus-server
  type: ClusterIP 
  ports:
    - port: 8080
      targetPort: 9090
```
The annotations in the above service YAML makes sure that the service endpoint is scrapped by Prometheus. The prometheus.io/port should always be the target port mentioned in service YAML

Create the service using the following command. Locate the prometheus-service.yaml file from my repo [yaml](https://github.com/mdditt2000/prometheus/blob/master/prometheus-service.yaml)
```
Kubectl create -f prometheus-service.yaml --namespace=monitoring
```

## About theses example / repo

* Coming soon [document](https://github.com/mdditt2000/kubernetes-1-18/blob/master/k8s%20cluster%20install/install%20guide/install-cluster.md)
* Coming soon [document](https://github.com/mdditt2000/kubernetes-1-18/blob/master/cis%201.14/QuickStartGuideCluster.md)
* Coming soon [document](https://github.com/mdditt2000/kubernetes-1-18/blob/master/cis%201.14/QuickStartGuideNode.md)
* Coming soon [document](https://github.com/mdditt2000/kubernetes-1-18/blob/master/cis%201.14/type-loadbalancer/QuickStartGuide.md)
* Coming soon [document](https://github.com/mdditt2000/kubernetes-1-18/blob/master/kubernetes-faq.md)