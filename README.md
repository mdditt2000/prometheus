# BIG-IP Metrics from Prometheus using Telemetry Streaming and Container Ingress Services

Prometheus is an open source monitoring framework. This user-guide covers setup of Prometheus for BIG-IP and CIS using F5 Telemetry Streaming. In this user-guide, Prometheus is deployed in Kubernetes and configured via a ConfigMap. BIG-IP is load balancing the management traffic to the Prometheus-UI via a Ingress automated via CIS as shown in the diagram below.

![diagram](https://github.com/mdditt2000/prometheus/blob/master/diagrams/2021-06-03_14-03-16.png)

Demo on YouTube [video](https://www.youtube.com/watch?v=IEAzvkRjWAE)

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

### Create a ConfigMap

Create a ConfigMap with all the prometheus scrape config and alerting rules, which will be mounted to the Prometheus container in /etc/prometheus as prometheus.yaml and prometheus.rules files

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

#### Exposing the Prometheus deployment as a service with F5 Load Balancer using Container Ingress Services

To access the Prometheus dashboard over a IP or a DNS name, you need to expose it as Kubernetes service.

### Create a file named prometheus-service.yaml

We will expose Prometheus using ClusterIP. ClusterIP allows the BIG-IP to forward traffic directly to the Prometheus pod bypassing kube-proxy. Use ClusterIP type, which will create a F5 BIG-IP load balancer and points it to the service
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
kubectl create -f prometheus-service.yaml -n monitoring
```

### Create a file named prometheus-ingress.yaml for Container Ingress Services

Create a Ingress for Container Ingress Services to configure F5 BIG-IP Locate the prometheus-deployment.yaml file from my repo [yaml](https://github.com/mdditt2000/prometheus/blob/master/prometheus-deployment.yaml)
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
The annotations in the above Ingress provides the public virtual-IP used to connect the prometheus-ui. BIG-IP will terminate SSL and work traffic to the pod on port 8080. You can also add additional security setting to the Ingress resource to prevent the prometheus-ui from web attacks.

Create the Ingress using the following command. Locate the prometheus-ingress.yaml file from my repo [yaml](https://github.com/mdditt2000/prometheus/blob/master/prometheus-ingress.yaml)

```
kubectl create -f prometheus-ingress.yaml -n monitoring
```
Once created, you can access the Prometheus dashboard using the virtual IP address

![Image of CRDs](https://github.com/mdditt2000/prometheus/blob/master/diagrams/2020-05-11_16-28-32.png)

## Configure BIG-IP Telemetry Streaming for Prometheus

Support for the Prometheus pull consumer is available in TS 1.12.0 and later

Install telemetry streaming rpm package on BIG-IP. Following link explains how to install the rpm on BIG-IP [Downloading and installing Telemetry Streaming](https://clouddocs.f5.com/products/extensions/f5-telemetry-streaming/latest/installation.html)

Prometheus support is available in Telemetry Streaming v1.12.0 and can download from the following link [rpm](https://github.com/F5Networks/f5-telemetry-streaming/releases/tag/v1.12.0)

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
        "systemPoller": [
            "My_Poller"
        ]
    },
    "metrics": {
        "class": "Telemetry_Pull_Consumer",
        "type": "Prometheus",
        "systemPoller": "My_Poller"
    }
}
```
The Prometheus Pull Consumer outputs the telemetry data according to the Prometheus data
model specification configured in Prometheus

## Create Prometheus user on BIG-IP

Create a user for basic_auth allowing Prometheus access to the metrics_path

### Configure Prometheus

Since we created a config map with all the prometheus scrape config and alerting rules, it be mounted to the Prometheus container in /etc/prometheus as prometheus.yaml and prometheus.rules files.

``` 
     - job_name: 'BIG-IP - TS'
        scheme: 'https'
        tls_config:
          insecure_skip_verify: true
        metrics_path: '/mgmt/shared/telemetry/pullconsumer/metrics'
        basic_auth:
          username: 'prometheus'
          password: <secret>
        static_configs:
        - targets: ['192.168.200.92']
```
Add BIG-IP - TS job_name to the config-map.yaml so it applies the configuration Prometheus.yaml

**Field description**

* scheme: How prometheus will connect to the polled deviceConfig
* tls_config: - Is where you  disable SSL certificate validation
* metrics path:  - the path used to retrieve metrics from the BIG-IP
* basic_auth: - credentials for Prometheus to authenticate to the BIG-IP
* static_configs: - Contains one or more targets for this prometheus job

Check the targets Prometheus dashboard to make sure Prometheus is able to pull BIG-IP

![Image of Target](https://github.com/mdditt2000/prometheus/blob/master/diagrams/2020-05-12_14-52-08.png)

There are many metrics available to graph or monitor. Example below virtualServers current connections. Use the label to graph the metric desired.
```
# HELP f5_clientside_curConns clientside.curConns
# TYPE f5_clientside_curConns gauge
f5_clientside_curConns{virtualServers="/k8s_AS3/Shared/ingress_10_192_75_107_80"} 0
f5_clientside_curConns{virtualServers="/k8s_AS3/Shared/ingress_10_192_75_107_443"} 8
f5_clientside_curConns{virtualServers="/k8s_AS3/Shared/ingress_10_192_75_108_80"} 0
```
Graph displaying concurrent connection

![Image of graph](https://github.com/mdditt2000/prometheus/blob/master/diagrams/2020-05-12_16-08-21.png)