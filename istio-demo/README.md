# istio-basic-demo
Basic istio installation & Demo

### Prerequisites

List any prerequisites or dependencies that need to be installed before can use the project.

- [Docker Desktop](https://www.docker.com/products/docker-desktop/)
- Kubernetes Cluster 4 CPU & 8 GB Memory

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

10. 
