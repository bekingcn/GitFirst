
[Link](https://github.com/VictoriaMetrics/helm-charts)

```SH
helm repo add vm https://victoriametrics.github.io/helm-charts/

helm repo update

helm search repo vm/


kubectl create namespace vm

# Install and ...
helm show values vm/victoria-metrics-cluster > values.yaml

# Export default values of victoria-metrics-cluster chart to file values.yaml:

# Test the installation with command:
helm install victoria-metrics vm/victoria-metrics-cluster -f values.yaml -n vm --debug --dry-run

# Install chart with command:
helm install victoria-metrics vm/victoria-metrics-cluster -f values.yaml -n vm

# Get the pods lists by running these commands:
kubectl get pods -A | grep 'victoria-metrics'

# or list all resorces of victoria-metrics

kubectl get all -n vm | grep victoria

# Get the application by running this commands:
helm list -f victoria-metrics -n vm

# See the history of versions of victoria-metrics application with command.
helm history victoria-metrics -n vm

```



### Uninstall
```SH
helm uninstall victoria-metrics -n vm
```