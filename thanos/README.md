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

# Installation kube-prometheus (`Prometheus-operator`,`Prometheus rules`,`Alertmanager`,`Grafana`)
https://github.com/kittisuw/thanos/blob/master/kube-prometheus/README.md
# Installation Thanos component(`query`,`store`,`rule`,`compact`)
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
