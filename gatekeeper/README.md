# Gatekeeper-basic-demo
Basic Gatekeeper Installation & Demo

### Prerequisites

List any prerequisites or dependencies that need to be installed before can use the project.

- [Docker Desktop](https://www.docker.com/products/docker-desktop/)
- Kubernetes Cluster **4 CPU & 8 GB Memory**

### Installation

Step-by-step instructions to install the Istio.

1. Install Gatekeeper
   
   ```sh
   kubectl apply -f https://raw.githubusercontent.com/open-policy-agent/gatekeeper/v3.15.0/deploy/gatekeeper.yaml
   ```
2. Load Balancer service restricted
```
apiVersion: templates.gatekeeper.sh/v1beta1
kind: ConstraintTemplate
metadata:
  name: loadbalancerconstraint
spec:
  crd:
    spec:
      names:
        kind: LoadBalancerConstraint
  targets:
    - target: admission.k8s.gatekeeper.sh
      rego: |
        package insomniacoder.constraint
        violation[{"msg": msg}] {
          input.review.kind.kind = "Service"
          input.review.operation = "CREATE"
          input.review.object.spec.type = "LoadBalancer"
          msg := "Service type LoadBalancer are restricted"
        }
---
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: LoadBalancerConstraint
metadata:
  name: loadbalancerconstraint
spec:
  match:
    kinds:
      - apiGroups: [""]
        kinds: ["Service"]
    namespaces:
      - "default"
---
kind: Service
apiVersion: v1
metadata:
  name: lb-service
  namespace: default
spec:
  type: LoadBalancer
  selector:
    app: opa-test
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
```


