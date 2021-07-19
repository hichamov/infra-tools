![image](images/prometheus-logo.png)

# Prometheus 

### Operator

We will be using the `coreos/prometheus` operator, all manifests and files can be found in the [following](https://github.com/prometheus-operator) repository.

We will keep our proper version in the current repository, under `manifests` directory.

Create a namespace called `monitoring`

```
kubectl create ns monitoring
kubectl config set-context $(kubectl config current-context) --namespace=monitoring
```

* To deploy the operator

```
kubectl apply -f manifests/operator
```

* Check the operator's status

```
kubectl get pods -n monitoring
```

### Prometheus

* Deploy  prometheus instance

```
kubectl apply -f manifests/prometheus-server/
```

This will create a statefulset for prometheus server, check the status of the server

```
kubectl get pods
```

The newly created prometheus instance will be blank, because ih has no configuration yet, to access the prometheus UI, run the following command, and access yout browser [http://127.0.0.1:9090]

```
kubectl port-forward --address 0.0.0.0 prometheus-k8s-0 9090
```

### kube-state-metrics

Before configuring the prometheus server, we need to deploy `kube-state-metrics` so it will expose a `/metrics` which will be scraped by prometheus

```
kubectl apply -f manifests/kube-state-metrics
```

### Cluster monitoring

We will create the configuration for the following components:

* kube-api-server
* kube-state-metrics
* kubelet
* node-exporter
  
```
kubectl apply -f manifests/cluster-monitoring
```

Then, delete the prometheus pod, so it will reload the configuration.

```
kubectl delete pod prometheus-k8s-0
```

And access the web UI again, go to `status/Targets`, you should see all components of the cluster.

If everything's up and running, we will deploy grafana

```
kubectl apply -f manifests/grafana
```

This deployment has pre-configured dashboards, which we will use to monitor our cluster.

NB: Due to the huge number of dashboards, we will create our custom dashboards.

### Node exporter

The lacking pieace is the `node-exporter`, this can be run inside kubernetes of directly in linux hosts, which is the prefferd way, running the node-exporter inside containers will cause a security issue (should not be run as root).

we will create an ansible script to deploy the node-exporter to all linux hosts

### Application monitoring

In this part, we will show how monitor an application running inside kubernetes.

We will use a simple python application that exposes a `/` and `/metrics` endpoints that exposes custom added metrics.

to deploy the application and its `servicemonitor`:

```
kubectl apply -f manifests/application/
```

NB: For simplicity purpose, other component's documentation will be add in [documentation](documentation) direcrtory.