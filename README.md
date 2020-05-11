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


## About theses example / repo

* Coming soon [document](https://github.com/mdditt2000/kubernetes-1-18/blob/master/k8s%20cluster%20install/install%20guide/install-cluster.md)
* Coming soon [document](https://github.com/mdditt2000/kubernetes-1-18/blob/master/cis%201.14/QuickStartGuideCluster.md)
* Coming soon [document](https://github.com/mdditt2000/kubernetes-1-18/blob/master/cis%201.14/QuickStartGuideNode.md)
* Coming soon [document](https://github.com/mdditt2000/kubernetes-1-18/blob/master/cis%201.14/type-loadbalancer/QuickStartGuide.md)
* Coming soon [document](https://github.com/mdditt2000/kubernetes-1-18/blob/master/kubernetes-faq.md)