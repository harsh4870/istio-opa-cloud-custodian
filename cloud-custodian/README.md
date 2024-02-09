# Cloud Custodian - C7n
Basic cloud custodian installation & Demo

### Prerequisites

List any prerequisites or dependencies that need to be installed before can use the project.

- [Docker Desktop](https://www.docker.com/products/docker-desktop/)
- Kubernetes Cluster **4 CPU & 8 GB Memory**

### Installation

Step-by-step instructions to install the **C7n**.

1. Download cloud custodian
   
   ```sh
   python3 -m venv custodian
   source custodian/bin/activate
   ```
   
2. Install specific plugin for **GCP**, **Azure**, **Oracle**, **AWS** (included in default download)
   ```sh
   pip install c7n_gcp
   ```
   
4. Creation your first policy, **gcp-policy.yaml**
   ```
   policies:
   - name: stop-instance-name-test
     description: |
       Stops all compute instances that are named "test"
     resource: gcp.instance
     filters:
       - type: value
         key: name
         value: test
     actions:
       - type: stop
   ```
  
5. Run the first policy
   ```
   custodian run --output-dir=./gcp-output gcp-policy.yaml
   ```
