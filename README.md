# Install knative serving cluster

* Install k3d local

```bash
# disable traefik, add registry
k3d cluster create knative -p "80:80@loadbalancer" --registry-create dev.local --k3s-arg="--disable=traefik@server:0" --kubeconfig-update-default --api-port 6550
```

* Apply crd's
```bash
kubectl apply -f https://github.com/knative/serving/releases/download/knative-v1.18.0/serving-crds.yaml
```

* Apply core serving
```bash
kubectl apply -f https://github.com/knative/serving/releases/download/knative-v1.18.0/serving-core.yaml
```

* Apply kourier 
```bash
kubectl apply -f https://github.com/knative/net-kourier/releases/download/knative-v1.18.0/kourier.yaml

# configure serving
kubectl patch configmap/config-network \
  --namespace knative-serving \
  --type merge \
  --patch '{"data":{"ingress-class":"kourier.ingress.networking.knative.dev"}}'
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
