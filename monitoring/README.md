# How to install Prometheus HA with Thanos for Kubernetes Monitoring
### Prerequisites
1. Object storage for side-car and Thanos (we're using ceph)
2. Install helm https://docs.aws.amazon.com/eks/latest/userguide/helm.html
3. Install Prometheus and grafana with prometheus-operator repo https://github.com/helm/charts/tree/master/stable/prometheus-operator
*Further development has moved to helmchart [kube-prometheus-stack](https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack)

4. File values.yaml after clone prometheus-operator repo

### Step by step
### THANOS-QUERY AND THANOS-SIDCAR
#### 1. create secret file(key) form s3 credential for thanos-sidecar connect persistent volumn name "thanos-objstore-config"
```
#kubectl -n monitoring create secret generic thanos-objstore-config –from-file=thanos.yaml=thanos-secret.yaml
```
#### 2.Add thanos-sidecar to prometheus-operator config and apply
```
vi /prometheus-operator/values.yaml
...
thanos:
      baseImage: quay.io/thanos/thanos
      version: v0.10.1
      objectStorageConfig:
        key: thanos.yaml
        name: thanos-objstore-config
...
#helm install prom-op stable/prometheus-operator — namespace monitoring -f values.yaml
```
#### 3.Create deployment of thanos-query
```
#kubectl apply -f querier-deployment.yaml
```
#### 4.Create ingress of thanos-sidcar
```
#kubectl apply -f thanos-sidecar-ingress.yaml
```
#### 5. create ServiceMonitor of thanos-query
```
#kubectl apply -f querier-service-monitor.yaml
```
### THANOS-PEERS
#### 6. create Service of thanos-peers
```
#kubectl apply -f thanos-peers-svc.yaml
```
### THANOS-STORE
#### 7. create ConfigMap of thanos-store
```
#kubectl apply -f thanos-store.yaml
```
### THANOS-COMPACT
#### 8. create StatefulSet of thanos-compactor
```
#kubectl apply -f thanos-compactor.yaml
```

#### 9. create Service of thanos-compactor
```
#kubectl apply -f thanos-compactor-service.yaml
```

#### 10. create ServiceMonitor of thanos-compactor
```
#kubectl apply -f thanos-compactor-service-monitor.yaml
```
### Check all service
Prometheus:
kubectl port-forward service/prometheus-operated 9090:9090 — namespace monitoring &
Grafana:
kubectl port-forward service/prometheus-operator-grafana 3000:80 — namespace monitoring
Thanos:
kubectl port-forward service/thanos-query 10902:9090 — namespace monitoring &

Add querier URL “thanos-query.monitoring.svc.cluster.local:9090” as a prometheus data source in grafana. Check the s3 bucket for data.