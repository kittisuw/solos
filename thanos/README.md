# Installation kube-prometheus-stack (prometheus-operator,alertmanager,Grafana)
### 1.Add repo
```
sudo snap install helm --classic
helm init
helm repo add prometheus-com https://prometheus-community.github.io/helm-charts
helm repo add stable https://kubernetes-charts.storage.googleapis.com/
helm repo update
helm repo list
```
### 2. Install kube-prometheus-stack from yaml
```
helm install -f values.yaml --create-namespace --namespace thanos promstack prometheus-com/kube-prometheus-stack
```
>using this command for export setting before edit helm inspect values prometheus-com/kube-prometheus-stack > values.yaml

### 3. Create secret for thanos sidecar
``` 
kubectl -n thanos create secret generic  thanos-storage-config --from-file=thanos-storage-config.yaml=thanos-storage-config.yaml 
kubectl get secret -n monitoring|grep thanos-storage-config
``` 
# Installation Thanos component(query,store,rule,compact)
###  4. Install
``` 
kubectl apply -f thanos-rule-configmap.yaml
kubectl apply -f thanos-query.yaml -f thanos-store.yaml -f thanos-rule.yaml -f thanos-compact.yaml
``` 

# Test by set port for and brow your public ip and port
``` 
(
kubectl -n thanos port-forward --address 0.0.0.0 svc/thanos-query 9090 &
kubectl -n thanos port-forward --address 0.0.0.0 service/thanos-rule 10902 &
kubectl -n thanos port-forward --address 0.0.0.0 service/promstack-kube-prometheus-alertmanager 9093 &
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
kubectl delete crd thanosrulers.monitoring.coreos.co
kubectl delete ns/thanos
``` 