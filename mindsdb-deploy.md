```SH
###FIRST CLONE THE WHOLE REPO cd helm/mindsdb   and then run this command   (namespace is option you can add it or if you remove it than it goes default)
cd ..

git clone https://github.com/mindsdb/mindsdb/

cd mindsdb/helm/mindsdb

minikube start --cpus=4 --memory=14G

helm upgrade -i \
  mindsdb mindsdb \
  --namespace mindsdb \
  --create-namespace
```