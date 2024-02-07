# istio-basic-demo
Basic istio Installation & Demo

### Prerequisites

List any prerequisites or dependencies that need to be installed before can use the project.

- [Docker Desktop](https://www.docker.com/products/docker-desktop/)
- Kubernetes Cluster **4 CPU & 8 GB Memory**

### Installation

Step-by-step instructions to install the Istio.

1. Download Istioctl
   
   ```sh
   wget -L https://istio.io/downloadIstio && chmod +x downloadIstio
   ```
3. Istio source download
   
   ```sh
   ./downloadIstio
   ```
4. Go to istio directory & start installation on your Local K8s cluster (Make sure kubeconfig points to local cluster)
   
   ```sh
   cd istio-* && istioctl install --set profile=demo
   ```
5. Verify the installation Namespace - **istio-system**

   ```sh
   kubectl get ns

   NAME              STATUS   AGE
   default           Active   48m
   istio-system      Active   3m43s
   kube-node-lease   Active   48m
   kube-public       Active   48m
   kube-system       Active   48m
   ```

7. Verify the external service Load Balancer
   
   ```sh
   kubectl -n istio-system get svc

   istio-ingressgateway   LoadBalancer   10.100.0.28     localhost     15021:30981/TCP,80:31747/TCP,443:30554/TCP,31400:31195/TCP,15443:31488/TCP   7m42s
   ```
   
8. Check if Gateway API exists else install
   ```
   kubectl get crd gateways.gateway.networking.k8s.io &> /dev/null || \
   { kubectl kustomize "github.com/kubernetes-sigs/gateway-api/config/crd?ref=v1.0.0" | kubectl apply -f -; }
   ```
   
9. Enable Istio sidecar injection by default in Namespace **Default**
   ```
   kubectl label namespace default istio-injection=enabled

   namespace/default labeled
   ```

10. Deploy HTTP-ECHO test application (Feel free to Skip)
    ```
    kubectl apply -f istio-demo/http-echo.yaml
    ```
11. Open browser [http://localhost/http-echo](http://localhost/http-echo)

### Install Book Info Demo App

1. Book info install app
   ```
   kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml
   ```

2. Install the gateway
   ```
   kubectl apply -f samples/bookinfo/networking/bookinfo-gateway.yaml
   ```

3. Open browser [http://localhost/productpage](http://localhost/productpage)

   Verify the **Round Robbin** on the browser, **Review** service's different version stars (red stars, black stars, no stars)
   
4. Apply **Default Destination** rules it uses the **subset**
   ```
   kubectl apply -f samples/bookinfo/networking/destination-rule-all.yaml
   ```

### Request Routing

1. Route request to **version 1** of each microservice 
```
kubectl apply -f samples/bookinfo/networking/virtual-service-all-v1.yaml
```
2. Open browser [http://localhost/productpage](http://localhost/productpage)

Verify request routing, on Browser no more stars in **review** service as all requests are going to **reviews:v1**

### Routing based on User identity

**Specifc user** routed to **specific version** of service. You can also implement **JWT Claim** based routing to Istio.  

 1. Route **user** request to **version 2** of **review** microservice (stars enable)
```
kubectl apply -f samples/bookinfo/networking/virtual-service-reviews-test-v2.yaml
```
2. Open browser [http://localhost/productpage](http://localhost/productpage)

3. **Sing In** jason/password, harsh/password

Current request flow

User Json -> productpage -> review:v2 -> rating
User Any  -> productpage -> review

Verify stars will be available product review section.

### Fault Injection

Add delay of few seconds from **review:v2 -> rating**

1. Apply the changes
```
kubectl apply -f samples/bookinfo/networking/virtual-service-all-v1.yaml
kubectl apply -f samples/bookinfo/networking/virtual-service-reviews-test-v2.yaml
kubectl apply -f samples/bookinfo/networking/virtual-service-ratings-test-delay.yaml
```

You've discovered a bug. The reviews service has failed due to hard-coded timeouts in the microservices.

The timeout between the reviews and ratings services is hard-coded at 10 seconds, thus as predicted, the 7-second delay you added has no effect on the reviews service. Between the product page and the reviews service, there is also a hard-coded timeout, which is 3 seconds plus 1 retry for a total of 6 seconds. The call to reviews on the product page therefore times out too soon and produces an error after 6 seconds.

### Circuit Breaking

Break the circuit if not working

1. Install the httpbin
```
kubectl apply -f samples/httpbin/httpbin.yaml
```

2. Create destination rule
```
kubectl apply -f - <<EOF
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: httpbin
spec:
  host: httpbin
  trafficPolicy:
    connectionPool:
      tcp:
        maxConnections: 1
      http:
        http1MaxPendingRequests: 1
        maxRequestsPerConnection: 1
    outlierDetection:
      consecutive5xxErrors: 1
      interval: 1s
      baseEjectionTime: 3m
      maxEjectionPercent: 100
EOF
```

3. Install **fortio** to hit requests
```
kubectl apply -f samples/httpbin/sample-client/fortio-deploy.yaml
```

4. Login to **fortio** POD
```
export FORTIO_POD=$(kubectl get pods -l app=fortio -o 'jsonpath={.items[0].metadata.name}')
```
5. Hit one request to **httpbin** to check reponse
```
kubectl exec "$FORTIO_POD" -c fortio -- /usr/bin/fortio curl -quiet http://httpbin:8000/get
```

6. Bombard request & break circuit
```
kubectl exec "$FORTIO_POD" -c fortio -- /usr/bin/fortio load -c 2 -qps 0 -n 20 -loglevel Warning http://httpbin:8000/get
```
Check reponse code 200 & 503

### Observability

 Kiali dashboard

 1. Install dashboard, prometheus, jaeger & service
 ```
 kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.20/samples/addons/kiali.yaml
 kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.20/samples/addons/prometheus.yaml
 kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.20/samples/addons/extras/zipkin.yaml
 kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.20/samples/addons/jaeger.yaml
 ```
 2. Open the dashboard
 ```
 istioctl dashboard kiali
 ```
