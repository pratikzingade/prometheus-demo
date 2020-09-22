# Prometheus and Grafana Demo

This is a demo to install prometheus and grafana on your kubernetes cluster. I have used minikube for demonsration purpose.

## Prerequisite

Note: this has been developed and tested on OS X. Others should be similar.

install [minikube](https://kubernetes.io/docs/tasks/tools/install-minikube/)

install [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)

Clone this repository to you machine.

## Installation

Make sure `minikube` is up.
 
#### Create the monitoring namespace:

```bash
kubectl apply -f monitoring-namespace.yaml
```

#### Create configMap for prometheus.

This lets us update the configuration regardless of image:

```bash
kubectl apply -f prometheus-config.yaml
```

#### Create prometheus deployment. 
Giving the pod a label with a key of `name` and a value of `prometheus` is used later to reference prometheus.

```bash
kubectl apply -f prometheus-deployment.yaml
```

You can see this by running `kubectl get deployments --namespace=monitoring`

#### Create prometheus service. 

```bash
kubectl apply -f prometheus-service.yaml
```
You can then view it by running `kubectl get services --namespace=monitoring prometheus -o yaml`.
One thing to note is that you will see something like nodePort: 30827 in the output.

In minikube you can run `minikube service --namespace=monitoring prometheus` and it will open a browser window accessing the service.


#### Deploy Grafana

Deploy grafana by creating its deployment and service by running 
```bash
kubectl apply -f grafana-deployment.yaml
kubectl apply -f grafana-service.yaml 
```
Go to grafana by running `minikube service --namespace=monitoring grafana`. Username\password is `admin\admin`

Create a new dashboard on grafana 
-   Click on the icon in the upper left of grafana and go to "Data Sources".
-   Click "Add data source".
-   For name, just use `prometheus`
-   Select "Prometheus" as the type
-   For the URL, we will actual use  [Kubernetes DNS service discovery](http://kubernetes.io/docs/user-guide/services/#dns). So, just enter  `http://prometheus:9090`. This means that grafana will lookup the  `prometheus`  service running in the same namespace as it on port 9090.

Create a New dashboard by clicking on the upper-left icon and selecting Dashboard->New. 
Click the green control and add a graph panel. Under metrics, select "prometheus" as the datasource. For the query, use  `sum(container_memory_usage_bytes) by (pod_name)`. Click save. This graphs the memory used per pod.

#### Prometheus Node Exporter Daemonset

Run `kubectl apply -f node-exporter-daemonset.yml` to create the daemon set. This will run an instance of this on every node. In minikube, there is only one node, but this concept scales to thousands of nodes.

After a minute or so, Prometheus will discover the node itself and begin collecting metrics from it. To create a dashboard in grafana using node metrics, follow the same procedure as before but use  `node_load1`  as the metric query. This will be the one minute load average of the nodes.