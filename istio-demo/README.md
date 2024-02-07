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

3. 
4. 
