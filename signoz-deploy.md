
```SH
minikube start --cpus=4 --memory=14G

# Add the SigNoz Helm repository to your client with name signoz by running the following command:
helm repo add signoz https://charts.signoz.io

# Verify that the repository is accessible to the Helm CLI by entering the following command:
helm repo list

# Use the kubectl create ns command to create a new namespace. SigNoz recommends you use platform for your new namespace:
kubectl create ns platform

# Run the following command to install the chart with the release name my-release and namespace platform:
helm --namespace platform install my-release signoz/signoz


# (Optional) To install a different version, you can use the --set flag to specify the version you wish to install. The following example command installs SigNoz version 0.8.0:

# helm --namespace platform install my-release signoz/signoz \
#   --set frontend.image.tag="0.8.0" \
#   --set queryService.image.tag="0.8.0"


# You can access SigNoz by setting up port forwarding and browsing to the specified port. The following kubectl port-forward example command forwards all connections made to localhost:3301 to <signoz-frontend-service>:3301:
export SERVICE_NAME=$(kubectl get svc --namespace platform -l "app.kubernetes.io/component=frontend" -o jsonpath="{.items[0].metadata.name}")
kubectl --namespace platform port-forward svc/$SERVICE_NAME 3301:3301


# Verify the Installation
kubectl -n platform get pods



# DELETE
helm --namespace platform uninstall my-release
```


### (Optional) Install a Sample Application and Generate Tracing Data

```SH
# Use the HotROD install script below to create a sample-application namespace and deploy HotROD application on it:
curl -sL https://github.com/SigNoz/signoz/raw/develop/sample-apps/hotrod/hotrod-install.sh \
  | HELM_RELEASE=my-release SIGNOZ_NAMESPACE=platform bash

# Using the kubectl -n sample-application get pods command, monitor the sample application pods. Wait for all the pods to be in running state:
kubectl -n sample-application get pods

# Use the following command to generate load:
kubectl --namespace sample-application run strzal --image=djbingham/curl \
  --restart='OnFailure' -i --tty --rm --command -- curl -X POST -F \
  'user_count=6' -F 'spawn_rate=2' http://locust-master:8089/swarm

# Browse to http://localhost:3301 and see the metrics and traces for your sample application.

# Use the following command to stop load generation:
kubectl -n sample-application run strzal --image=djbingham/curl \
  --restart='OnFailure' -i --tty --rm --command -- curl \
  http://locust-master:8089/stop


# DELETE
curl -sL https://github.com/SigNoz/signoz/raw/develop/sample-apps/hotrod/hotrod-delete.sh \
  | HELM_RELEASE=my-release SIGNOZ_NAMESPACE=platform bash
```



### hotrod-install.sh - Sample Application installation script
```SH
#!/bin/bash

cd "$(dirname "${BASH_SOURCE[0]}")";

# Namespace to install sample app
HOTROD_NAMESPACE=${HOTROD_NAMESPACE:-"sample-application"}
SIGNOZ_NAMESPACE="${SIGNOZ_NAMESPACE:-platform}"

# HotROD's docker image
if [[ -z $HOTROD_IMAGE ]]; then
    HOTROD_REPO="${HOTROD_REPO:-jaegertracing/example-hotrod}"
    HOTROD_TAG="${HOTROD_TAG:-1.30}"
    HOTROD_IMAGE="${HOTROD_REPO}:${HOTROD_TAG}"
fi

# Locust's docker image
if [[ -z $LOCUST_IMAGE ]]; then
    LOCUST_REPO="${LOCUST_REPO:-signoz/locust}"
    LOCUST_TAG="${LOCUST_TAG:-1.2.3}"
    LOCUST_IMAGE="${LOCUST_REPO}:${LOCUST_TAG}"
fi

# Helm release name
HELM_RELEASE="${HELM_RELEASE:-my-release}"

# Otel Collector service address
if [[ -z $JAEGER_ENDPOINT ]]; then
    if [[ "$HELM_RELEASE" == *"signoz"* ]]; then
        JAEGER_ENDPOINT="http://${HELM_RELEASE}-otel-collector.${SIGNOZ_NAMESPACE}.svc.cluster.local:14268/api/traces"
    else
        JAEGER_ENDPOINT="http://${HELM_RELEASE}-signoz-otel-collector.${SIGNOZ_NAMESPACE}.svc.cluster.local:14268/api/traces"
    fi
fi

# Create namespace for sample application if does not exist
kubectl create namespace "$HOTROD_NAMESPACE" --save-config --dry-run -o yaml 2>/dev/null | kubectl apply -f -

# Setup sample apps into specified namespace
kubectl apply --namespace="${HOTROD_NAMESPACE}" -f <( \
    (cat hotrod-template.yaml 2>/dev/null || curl -sL https://github.com/SigNoz/signoz/raw/develop/sample-apps/hotrod/hotrod-template.yaml) | \
        HOTROD_NAMESPACE="${HOTROD_NAMESPACE}" \
        HOTROD_IMAGE="${HOTROD_IMAGE}" \
        LOCUST_IMAGE="${LOCUST_IMAGE}" \
        JAEGER_ENDPOINT="${JAEGER_ENDPOINT}" \
        envsubst \
    )

if [ $? -ne 0 ]; then
    echo "❌ Failed to deploy HotROD sample application"
else
    echo "✅ Successfully deployed HotROD sample application"
fi
```