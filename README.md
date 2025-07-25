This folder contains configuration files for chronocollector to collect metrics from the following services:
- cadvisor
- kube-state-metrics
- node-exporter
- dcgm-exporter

and 

- pods that are annotated with prefix "prometheus.io/scrape:true"

The configuration assumes chronocollector is deployed into namespace: chronosphere. Here are the steps:

## Create secrets from command line
```sh
kubectl create secret generic chronosphere-secret \
  --from-literal=api-token=CHANGE_ME \
  --from-literal=address=CHANGE_ME.chronosphere.io:443 \
  --namespace=<namespace-if-not-default. e.g. chronosphere>
```

## Install custom resource definition for service monitors
```sh
kubectl apply -f https://raw.githubusercontent.com/prometheus-community/helm-charts/e46dc6360b6733299452c8fd65d304004484de79/charts/kube-prometheus-stack/crds/crd-servicemonitors.yaml
```
For additional details, see [online doc](https://docs.chronosphere.io/ingest/metrics-traces/collector/discover#servicemonitors)

## Install Kube-State-Metrics
If kube-state-metrics is not already installed (e.g. via the kube-prometheus stack), it can be installed as follows:
```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install kube-state-metrics prometheus-community/kube-state-metrics -n <namespace-if-not-default>
```

## Install Node Exporter
If node_exporter is not already installed, it can be installed as follows:
```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install prometheus-node-exporter prometheus-community/prometheus-node-exporter  -n <namespace-if-not-default>
```

## Install Chronocollector Daemonset and Deployment
To set labels for all metrics collected by the collector, modify the labels.defaults section of the configmap.
```sh
kubectl apply -f chronocollector-daemonset.yaml -n <namespace-if-not-default. e.g. chronosphere>
```

```sh
kubectl apply -f chronocollector-deployment.yaml -n <namespace-if-not-default. e.g. chronosphere>
```
For large clusters, Chronocollector deployment is used to scrape kube-state-metrics to conserve resource and improve performance. For details see [online doc](https://docs.chronosphere.io/ingest/metrics-traces/collector/discover/monitor-kubernetes#discover-and-scrape-kube-state-metrics)

## Configure Service Monitors
Service monitor definition assumes that services are deployed using Prometheus Community Helm charts.

```
kubectl apply -f service-monitors.yaml -n <namespace-if-not-default. e.g. chronosphere>
```