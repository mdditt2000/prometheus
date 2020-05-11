# Prometheus Example - BIGIP with Telemetry Streaming and Container Ingress Services
Prometheus is an open source monitoring framework. Explaining Prometheus is out of the scope of this repo. In this repo, I will guide you to setup Prometheus on a BIGIP and use Telemetry Streaming and collect metrics

## How to Setup Prometheus Monitoring On Kubernetes Cluster

### Create a Namespace and ClusterRole
Create a Kubernetes namespace for all our monitoring components
```
kubectl create namespace monitoring
```
Create a file named clusterRole.yaml and copy the content of this. Locate the yaml file from my repo clusterRole.yaml [yaml](https://github.com/mdditt2000/prometheus/blob/master/clusterRole.yaml)
```
kubectl create -f clusterRole.yaml
```


## About theses example / repo

* Coming soon [document](https://github.com/mdditt2000/kubernetes-1-18/blob/master/k8s%20cluster%20install/install%20guide/install-cluster.md)
* Coming soon [document](https://github.com/mdditt2000/kubernetes-1-18/blob/master/cis%201.14/QuickStartGuideCluster.md)
* Coming soon [document](https://github.com/mdditt2000/kubernetes-1-18/blob/master/cis%201.14/QuickStartGuideNode.md)
* Coming soon [document](https://github.com/mdditt2000/kubernetes-1-18/blob/master/cis%201.14/type-loadbalancer/QuickStartGuide.md)
* Coming soon [document](https://github.com/mdditt2000/kubernetes-1-18/blob/master/kubernetes-faq.md)