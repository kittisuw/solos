# KUBE-PROMETHEUS
```
# Clone the Prometheus Operator repository
git clone https://github.com/kittisuw/solos.git
cd kube-prometheus

# Create the namespace and CRDs, and then wait for them to be availble before creating the remaining resources
kubectl create -f manifests/setup
until kubectl get servicemonitors --all-namespaces ; do date; sleep 1; echo ""; done
kubectl create -f manifests/

# Get created resources
kubectl get all -n thanos

# Acces to Prometheus dashboard
kubectl --namespace thanos port-forward --address 0.0.0.0 svc/prometheus-k8s 9090

# Acces to Grafana dashboard
kubectl --namespace thanos port-forward --address 0.0.0.0 svc/grafana 3000

# Acces to Alert Manager dashboard
kubectl --namespace thanos port-forward --address 0.0.0.0 svc/alertmanager-main 9093
```
