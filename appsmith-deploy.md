### install
```SH

minikube start --cpus=4 --memory=14G
# enable ingress
minikube addons enable ingress

# appsmith
helm repo add stable-appsmith http://helm.appsmith.com
helm repo update
helm install am-demo stable-appsmith/appsmith
# helm install am-demo stable-appsmith/appsmith --set ingress.enabled=true --set ingress.annotations."kubernetes\.io/ingress\.class"=nginx --set service.type=ClusterIP

# upgrade
helm upgrade am-demo stable-appsmith/appsmith --set ingress.enabled=true --set ingress.annotations."kubernetes\.io/ingress\.class"=nginx --set service.type=ClusterIP

nohup kubectl port-forward --address 0.0.0.0 svc/am-demo-appsmith 18880:80  > pf18880.out &

```


### test env
```SH
# postgresql
helm repo add cetic https://cetic.github.io/helm-charts
helm repo update
helm install test-pg cetic/postgresql

# nohup kubectl port-forward --address 0.0.0.0 svc/test-pg-postgresql 5432:5432  > pf5432.out &

# pg client
kubectl run postgres-client --rm --tty -i --restart='Never' --image postgres:11 --env="PGPASSWORD=postgres" --command -- psql -h test-pg-postgresql -U postgres
```



### uninstall
```SH
helm delete --purge test-pg

helm uninstall am-demo
```