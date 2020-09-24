# How to install Prometheus HA with Thanos for Kubernetes Monitoring
### Prerequisites
1. Object storage for side-car and Thanos (we're using ceph)
2. Install helm https://docs.aws.amazon.com/eks/latest/userguide/helm.html
3. Install Prometheus and grafana with prometheus-operator repo https://github.com/helm/charts/tree/master/stable/prometheus-operator
4. File values.yaml after clone prometheus-operator repo

### Step by step
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
#### 3.Create deployment for thanos-query
```
#kubectl apply -f querier-deployment.yaml
```
#### 4.Create ingress for thanos-sidcar
```
#kubectl apply -f thanos-sidecar-ingress.yaml
```
#### 5. create ServiceMonitor
```
#kubectl apply -f querier-service-monitor.yaml
```
#### 6. create Service for thanos-peers
```
#kubectl apply -f thanos-peers-svc.yaml
```

#### 7. create ConfigMap for thanos-store
```
#kubectl apply -f thanos-store.yaml
```

#### 8. create StatefulSet of thanos-compactor
```
# kubectl apply -f thanos-compactor.yaml
```

#### 9. create Service of thanos-compactor
```
# kubectl apply -f thanos-compactor-service.yaml
```
