**NOte: the editor works bad. it better to use jupyter with it**


```SH
###FIRST CLONE THE WHOLE REPO cd helm/mindsdb   and then run this command   (namespace is option you can add it or if you remove it than it goes default)
cd ..

git clone https://github.com/mindsdb/mindsdb/

cd mindsdb/helm

minikube start --cpus=4 --memory=14G

helm upgrade -i \
  mindsdb mindsdb \
  --namespace mindsdb \
  --create-namespace


# sttus...
kubectl -n mindsdb get pods

# disk view
df -h


export MINDSDB_SERVICE_NAME=$(kubectl get svc --namespace mindsdb -l "app.kubernetes.io/name=mindsdb" -o jsonpath="{.items[0].metadata.name}")
kubectl --namespace mindsdb port-forward svc/$MINDSDB_SERVICE_NAME 47334:47334
```


```SQL


CREATE MODEL 
  mindsdb.home_model 
FROM files
  (SELECT * FROM home_table)
PREDICT median_house_value;  


```