![Overview](https://github.com/kittisuw/solos/blob/master/thanos/img/overview.jpg)
# Service port overview
Service | Internal port | Public port
------------ | ------------- | -------------
Prometheus | 1234 | 1234
Grafana | 1234 | 1234
Alertmanager| 1234 | 1234
Thanos-sidecar | 10901(gRPC) | 1234
Thanos-query | 10901(gRPC)  | 1234
Thanos-store | 10901(gRPC)  | 1234
Thanos-rule | 10901(gRPC)  | 1234
Thanos-compact | 10901(gRPC)  | 1234

# Installation kube-prometheus-stack (`Prometheus-operator`,Prometheus rules,Alertmanager,Grafana)
1.Install helm and add helm-charts
```
sudo snap install helm --classic
helm init
helm repo add prometheus-com https://prometheus-community.github.io/helm-charts
helm repo add stable https://kubernetes-charts.storage.googleapis.com/
helm repo update
helm repo list

NAME            URL                                               
prometheus-com  https://prometheus-community.github.io/helm-charts
stable          https://kubernetes-charts.storage.googleapis.com/ 
```
2.Install kube-prometheus-stack (namespace:thanos name:promstack)
```
helm install -f values.yaml --create-namespace --namespace thanos promstack prometheus-com/kube-prometheus-stack
```
>using this command for export setting before edit helm inspect values prometheus-com/kube-prometheus-stack > values.yaml

3.Create S3 secret that you config in values.yaml(kube-prometheus-stack)
```
kubectl -n thanos create secret generic  thanos-storage-config --from-file=thanos-storage-config.yaml=thanos-storage-config.yaml 
kubectl get secret -n monitoring|grep thanos-storage-config
```
# Installation Thanos component(query,store,rule,compact)
4.Install
``` 
kubectl apply -f thanos-rule-configmap.yaml
kubectl apply -f thanos-query.yaml -f thanos-store.yaml -f thanos-rule.yaml -f thanos-compact.yaml
``` 
### Set port forward and Test Thanos-query,Thanos-rule,Alertmanager via browser
``` 
(
kubectl -n thanos port-forward --address 0.0.0.0 svc/thanos-query 9090 &
kubectl -n thanos port-forward --address 0.0.0.0 service/thanos-rule 10902 &
kubectl -n thanos port-forward --address 0.0.0.0 service/promstack-kube-prometheus-alertmanager 9093 &
kubectl -n thanos port-forward --address 0.0.0.0 service/promstack-grafana 80 & 
)
``` 

# Uninstallations
``` 
helm uninstall promstack -n thanos
kubectl delete crd prometheuses.monitoring.coreos.com
kubectl delete crd prometheusrules.monitoring.coreos.com
kubectl delete crd servicemonitors.monitoring.coreos.com
kubectl delete crd podmonitors.monitoring.coreos.com
kubectl delete crd alertmanagers.monitoring.coreos.com
kubectl delete crd thanosrulers.monitoring.coreos.com
kubectl delete ns/thanos
``` 