# Install knative on k3d

Follow the [installation documentation](https://knative.dev/docs/install/operator/knative-with-operators/) to install knative  operator using helm.

* Install k3d local first

```bash
# disable traefik
k3d cluster create knative -p "80:80@loadbalancer" --k3s-arg="--disable=traefik@server:0" --kubeconfig-update-default
```


* Installing the operator
```bash
helm repo add knative-operator https://knative.github.io/operator
helm install knative-operator --create-namespace --namespace knative-operator knative-operator/knative-operator
```

* Create the Serving custom resource with `kourier` networking layer

```bash
kubectl apply -f kn-serving-kourier.yaml

namespace/knative-serving created
knativeserving.operator.knative.dev/knative-serving created
```

* Fetch external IP

```bash
kubectl --namespace kourier-system get service kourier

NAME      TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)                      AGE
kourier   LoadBalancer   10.43.230.18   172.18.0.2    80:31713/TCP,443:32571/TCP   23m
```

* configure sample domain
```bash
kubectl patch configmap/config-domain \
      --namespace knative-serving \
      --type merge \
      --patch '{"data":{"example.com":""}}'
```

* apply example knative service
```bash
kubectl apply --filename knativeservice.yaml
```

* get knative service url
```bash
kubectl get ksvc                                                         
NAME            URL                                        LATESTCREATED         LATESTREADY           READY   REASON
helloworld-go   http://helloworld-go.default.example.com   helloworld-go-00001   helloworld-go-00001   True    
```

* test service with curl using External LoadBalancer IP from above and Host header from URL above
```bash
curl -v -H "Host: helloworld-go.default.example.com" 172.18.0.2

# Resonse:
Hello Hello Knative Serving is up and running with Kourier!!!
```

* or, add host to /etc/hosts and call the service:

```bash
curl -v helloworld-go.default.example.com
```


