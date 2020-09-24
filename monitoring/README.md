# How to install Prometheus HA with Thanos for Kubernetes Monitoring
### Prerequisites
1. Object storage for side-car and Thanos (we're using ceph)
2. Install helm https://docs.aws.amazon.com/eks/latest/userguide/helm.html
3. Install Prometheus and grafana with prometheus-operator repo https://github.com/helm/charts/tree/master/stable/prometheus-operator
4. File values.yaml after clone prometheus-operator repo

### Step by step
1. create secret for thanos-sidecar connect persistent volumn name "thanos-objstore-config"
```
kubectl -n monitoring create secret generic thanos-objstore-config –from-file=thanos.yaml=thanos-secret.yaml
```


### Add new name space
```
kubectl apply -f namespace.yml
```