![Overview](https://github.com/kittisuw/solos/blob/master/thanos/img/overview.jpg)
# Service port overview
Service | Internal port | Public port
------------ | ------------- | -------------
**Prometheus** | 1234 | 1234
**Grafana** | 1234 | 1234
**Alertmanager**| 1234 | 1234
**Thanos-sidecar** | 10901(gRPC) | 1234
**Thanos-query** | 10901(gRPC)  | 1234
**Thanos-store** | 10901(gRPC)  | 1234
**Thanos-rule** | 10901(gRPC)  | 1234
**Thanos-compact** | 10901(gRPC)  | 1234

# prerequisite
1.Clone repo
```
git clone https://github.com/kittisuw/solos.git
```
# Installation kube-prometheus (`Prometheus-operator include Thanos-sidcar`,`Prometheus rules`,`Alertmanager`,`Grafana`)
2.create namespace:thanos and apply S3 secret for object storage that thanos-store and thanos-sidcar using
``` 
kubectl create namespace thanos
cd thanos
kubectl apply -f thanos-storage-config.yaml
``` 
3.Install kube-prometheus
```
cd ../kube-prometheus
kubectl create -f manifests/setup
until kubectl get servicemonitors --all-namespaces ; do date; sleep 1; echo ""; done
kubectl create -f manifests/
``` 
4.check kube-prometheus service
``` 
kubectl get all -n thanos
```
> https://github.com/prometheus-operator/kube-prometheus
# Installation Thanos rule config map and main component(`query`,`store`,`rule`,`compact`)
2.
``` 
kubectl apply -f thanos-rule-configmap.yaml
kubectl apply -f thanos-query.yaml -f thanos-store.yaml -f thanos-rule.yaml -f thanos-compact.yaml
``` 
### Set port-forward or ingress for view Thanos-query,Thanos-rule,Thanos-Alertmanager,Grafana via browser
``` 
(
kubectl -n thanos port-forward --address 0.0.0.0 svc/thanos-query 9090 &
kubectl -n thanos port-forward --address 0.0.0.0 service/thanos-rule 10902 &
kubectl -n thanos port-forward --address 0.0.0.0 service/alertmanager-main 9093 &
kubectl -n thanos port-forward --address 0.0.0.0 service/grafana 80 & 
)
```
or set ingress
```
k apply -f thanos-ing.yaml
k describe ing/thanos-ing -n thanos
```
# Uninstallations
1. kube-prometheus
```
cd kube-prometheus
kubectl delete -f manifests/
```
2. thanos
```
kubectl delete all --all -n thabos
```
