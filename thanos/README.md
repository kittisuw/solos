# Installation kube-prometheus-stack
### 1.Add repo
```
#sudo snap install helm --classic
#helm init
#helm repo add prometheus-com https://prometheus-community.github.io/helm-charts
#helm repo add stable https://kubernetes-charts.storage.googleapis.com/
#helm repo update
#helm repo list
```
# 2. Install kube-prometheus-stack
```
#helm install -f values.yaml --create-namespace --namespace thanos promstack prometheus-com/kube-prometheus-stack
```


``` 
#kubectl -n thanos create secret generic  thanos-storage-config --from-file=thanos-storage-config.yaml=thanos-storage-config.yaml 
#kubectl get secret -n monitoring|grep thanos-storage-config
``` 
