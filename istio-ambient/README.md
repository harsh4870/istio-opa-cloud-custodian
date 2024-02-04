# istio-ambient-demo
Istio Ambient Installation & Demo

### Prerequisites

List any prerequisites or dependencies that need to be installed before can use the project.

- [Docker Desktop](https://www.docker.com/products/docker-desktop/)
- Kubernetes Cluster **4 CPU & 8 GB Memory**

### Installation

Step-by-step instructions to install the Istio.

1. Download Istioctl
   
   ```sh
   curl -L https://istio.io/downloadIstio | ISTIO_VERSION=1.21.0-beta.1 sh -
   ```
2. Untar the downloaded file - istio-1.21.0-beta.1-linux-arm64.tar.gz

4. Go to site directory & start installation on your Local K8s cluster (Make sure kubeconfig points to local cluster)
   
   ```sh
   cd istio-1.21.0-beta.1 && export PATH=$PWD/bin:$PATH && istioctl install --set profile=ambient --set "components.ingressGateways[0].enabled=true" --set "components.ingressGateways[0].name=istio-ingressgateway"  --skip-confirmation 
   ```
5. Verify the installation in Namespace - **istio-system**

   ```sh
   kubectl -n istio-system get pods

   NAME                                    READY   STATUS    RESTARTS   AGE
   istio-cni-node-dlr85                    1/1     Running   0          58s
   istio-ingressgateway-689f9d6fb4-f8jpk   1/1     Running   0          59s
   istiod-556d7d4cf5-5477v                 1/1     Running   0          63s
   ztunnel-d4mrw                           1/1     Running   0          54s
   ```

7. Verify the external service Load Balancer
   
   ```sh
   kubectl -n istio-system get svc

   istio-ingressgateway   LoadBalancer   10.102.102.71    localhost     15021:31777/TCP,80:31787/TCP,443:32736/TCP   105s
   ```
   
8. Check if Gateway API exists else install
   ```
   kubectl get crd gateways.gateway.networking.k8s.io &> /dev/null || \
   { kubectl kustomize "github.com/kubernetes-sigs/gateway-api/config/crd?ref=v1.0.0" | kubectl apply -f -; }
   ```
   
9. Enable Istio ambient mode in Namespace **Default**
   ```
   kubectl label namespace default istio.io/dataplane-mode=ambient

   namespace/default labeled
   ```

10. Deploy HTTP-ECHO test application
    ```
    kubectl apply -f istio-demo/http-echo.yaml
    ```
11. Open browser [http://localhost/http-echo](http://localhost/http-echo)
