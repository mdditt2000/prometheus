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

#### Using Kubectl port forwarding

Follow the using Kubectl port forwarding steps documented at devopscube.com [link](https://devopscube.com/setup-prometheus-monitoring-on-kubernetes/)

#### Exposing the Prometheus deployment as a service with F5 Load Balancer using Container Ingress Services

To access the Prometheus dashboard over a IP or a DNS name, you need to expose it as Kubernetes service.

### Create a file named prometheus-service.yaml
We will expose Prometheus using ClusterIP. ClusterIP allows the BIGIP to forward traffic directly to the Prometheus pod bypassing kube-proxy. Use ClusterIP type, which will create a F5 BIGIP load balancer and points it to the service
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
### Create a file named prometheus-ingress.yaml for Container Ingress Services
Create a Ingress for Container Ingress Services to configure F5 BIGIP Locate the prometheus-deployment.yaml file from my repo [yaml](https://github.com/mdditt2000/prometheus/blob/master/prometheus-deployment.yaml)
```
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata: 
  annotations: 
    virtual-server.f5.com/http-port: "443"
    ingress.kubernetes.io/allow-http: "false"
    ingress.kubernetes.io/ssl-redirect: "true"
    virtual-server.f5.com/ip: "10.192.75.107"
  name: prometheus-ui
  namespace: monitoring
spec: 
  backend: 
    serviceName: prometheus-service
    servicePort: 8080
  tls: 
    - 
      hosts: ~
      secretName: /Common/clientssl
```
The annotations in the above Ingress provides the public virtual-IP used to connect the prometheus-ui. BIGIP will terminate SSL and work traffic to the pod on port 8080. You can also add additional security setting to the Ingress resource to prevent the prometheus-ui from web attacks.

Create the Ingress using the following command. Locate the prometheus-ingress.yaml file from my repo [yaml](https://github.com/mdditt2000/prometheus/blob/master/prometheus-ingress.yaml)
```
Kubectl create -f prometheus-ingress.yaml --namespace=monitoring
```
Once created, you can access the Prometheus dashboard using the virtual IP address 

![Image of CRDs](https://github.com/mdditt2000/prometheus/blob/master/diagrams/2020-05-11_16-28-32.png)

## Configure BIGIP Telemetry Streaming for Prometheus

Support for the Prometheus pull consumer is available in TS 1.12.0 and later

Install telemetry streaming rpm package on BIGIP. Following link explains how to install the rpm on BIGIP [Downloading and installing Telemetry Streaming](https://clouddocs.f5.com/products/extensions/f5-telemetry-streaming/latest/installation.html)

Since Prometheus support for Telemetry Streaming has not being released you can find the [rpm](https://github.com/mdditt2000/prometheus/blob/master/rpm/f5-telemetry-1.12.0-1.noarch.rpm)

## Configure Telemetry Streaming declaration
This example shows how to use the Prometheus pull consumer. For this pull consumer, the type
must be Prometheus in the Pull Consumer class as shown
```
{
    "class": "Telemetry",
    "My_Poller": {
        "class": "Telemetry_System_Poller",
        "interval": 0
    },
    "My_System": {
        "class": "Telemetry_System",
        "enable": "true",
        "systemPoller": "My_Poller"
    },
    "My_Pull_Consumer": {
        "class": "Telemetry_Pull_Consumer",
        "type": "Prometheus",
        "systemPoller": "My_Poller"
    }
}
```
The Prometheus Pull Consumer outputs the telemetry data according to the Prometheus data
model specification configured in Prometheus

### Configure Prometheus
Since we created a config map with all the prometheus scrape config and alerting rules, it be mounted to the Prometheus container in /etc/prometheus as prometheus.yaml and prometheus.rules files.

``` 
     - job_name: 'BIGIP - TS'
        scheme: 'https'
        tls_config:
          insecure_skip_verify: true
        metrics_path: '/mgmt/shared/telemetry/pullconsumer/metrics'
        basic_auth:
          username: 'admin'
          password: 'admin'
        static_configs:
        - targets: ['192.168.200.92']
```
Add BIGIP - TS job to the config-map.yaml so its mounts to the Prometheus container.



## About theses example / repo

* Coming soon [document](https://github.com/mdditt2000/kubernetes-1-18/blob/master/k8s%20cluster%20install/install%20guide/install-cluster.md)
* Coming soon [document](https://github.com/mdditt2000/kubernetes-1-18/blob/master/cis%201.14/QuickStartGuideCluster.md)
* Coming soon [document](https://github.com/mdditt2000/kubernetes-1-18/blob/master/cis%201.14/QuickStartGuideNode.md)
* Coming soon [document](https://github.com/mdditt2000/kubernetes-1-18/blob/master/cis%201.14/type-loadbalancer/QuickStartGuide.md)
* Coming soon [document](https://github.com/mdditt2000/kubernetes-1-18/blob/master/kubernetes-faq.md)