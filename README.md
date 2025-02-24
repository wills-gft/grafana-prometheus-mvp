# An MVP for running Prometheus in GKE

Kubernetes in Google Kubernetes Engine, running Prometheus (Latest), scraping Kubernetes, itself and data from the nodes in the cluster.

Now with Persistent Volume on Prometheus! (so you can `delete` the Prometheus pod without losing data..)

Now with Latest! (You still have to configure the Datasources manually because.. I'm in the making of doing this.)



## Requirements

`kubectl`, `gcloud` and (of course..) [rights to create the RBAC resources in the GKE cluster](https://coreos.com/operators/prometheus/docs/latest/troubleshooting.html), see below:

```
kubectl create clusterrolebinding myname-cluster-admin-binding --clusterrole=cluster-admin --user=me@domain.com
```

NB: remember to use the email associated with your GCP account

## Setup (minimal instructions)

1. Have access to a GKE cluster with `kubectl`
1. Run `setup.sh`
1. Wait about 10-15 minutes before the loadbalancers start serving traffic
1. Configure Prometheus `data source` in Grafana



## Cleanup

Run `teardown.sh`

## Setup (detailed instructions)

### 1. Create a cluster on GKE
```
$ gcloud container clusters create my-prometheus-cluster
```

### 2. Create the `RBAC` resources from the `rbac/` folder
```
$ kubectl create -f rbac/service-account-prometheus.yaml
$ kubectl create -f rbac/clusterrole-prometheus.yaml
$ kubectl create -f rbac/clusterrolebinding-prometheus.yaml
```

Optional: Create the Node Exporter `daemonset` (if you want node-metrics like CPU-usage, etc.)
```
$ kubectl create -f node-exporter/daemonset-node-exporter.yaml
```

### 3. Create the `configmap` that we're going to mount into the Prometheus `deployment`
```
$ kubectl create -f prometheus/configmap-prometheus.yaml
```

### 4. Create the Prometheus `deployment` (with a `persistent volume`)
```
$ kubectl create -f prometheus/pv-deployment-prometheus.yaml
```

"old" deployment, w/o persistent volume: (or swap the commented lines in `setup.sh`, `teardown.sh`)
```
$ kubectl create -f prometheus/deployment-prometheus.yaml
```

### 5. Create the Prometheus Service, exposing the deployment as a `NodePort` since GKE ingress' require this
```
$ kubectl create -f networking/service-prometheus.yaml
```

### 6. Create the `ingress` loadbalancer
```
$ kubectl create -f networking/ingress-prometheus.yaml
```

### 7. Plenty of patience and Bob's your uncle

_Use the following command to find the address Prometheus is being served on,_
```
$ kubectl get svc -n istio-system 
```

Optional: Create the Grafana resources from the `grafana/` folder, (you still have to add the datasource, for `URL` use the ip from the Prometheus `service` e.g. http://10.59.253.76:9090, and `Access: Proxy`)
```
$ kubectl create -f grafana/deployment-grafana.yaml
$ kubectl create -f grafana/service-grafana.yaml
$ kubectl create -f grafana/ingress-grafana.yaml
```

Use the following command to find the address Grafana is being served on,_
```
$ kubectl get svc -n istio-system 
```

