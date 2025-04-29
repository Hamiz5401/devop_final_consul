# Consul Crash Course: Demo Project

This repository is a companion demo for the **Consul Crash Course** video on YouTube. It demonstrates deploying an online store across two Kubernetes clusters with HashiCorp Consul for service discovery and mesh networking. Infrastructure is provisioned using **Terraform**, and deployments are managed with **Kubernetes** and **Helm**.

---

## 🌍 Project Structure

### Terraform
Used to provision infrastructure (e.g., EKS and Linode clusters).

- `providers.tf`: Specifies which cloud providers used by Terraform.
- `variables.tf`: Defines configurable variables.
- `main.tf`: Contains the actual infrastructure definitions and resource attributes.

### Kubernetes
Used to deploy the demo application and Consul components.

- `config.yaml`: Creates deployments and services for the online store (used **before** Consul is deployed).
- `config-consul.yaml`: Similar to `config.yaml` but with **Consul-specific annotations** for service mesh integration (used **after** Consul is deployed).
- `consul-values.yaml`: Helm chart values for installing Consul on the Kubernetes cluster.
- `consul-mesh-gateway.yaml`: Defines a Consul mesh gateway to connect services between two clusters.
- `exported-service.yaml`: Exports the `shippingservice` from the secondary (Linode) cluster to the primary (EKS) cluster.
- `service-resolver.yaml`: Sets the imported `shippingservice` as a failover in the primary cluster.

---

## ⚙️ Terraform Usage

```bash
# Initialize the Terraform project and download provider plugins
terraform init

# Preview the planned changes
terraform plan

# Apply changes with variable file (with preview)
terraform apply -var-file terraform.tfvars

# Apply changes automatically (no prompt)
terraform apply -var-file terraform.tfvars -auto-approve

# Destroy all provisioned infrastructure
terraform destroy

# View current Terraform-managed resources
terraform state list
```

## AWS Cluster

```bash
# Configure AWS CLI with credentials
aws configure

# List available EKS clusters
aws eks list-clusters --region eu-central-1 --output table --query 'clusters'

# View VPC config and API access settings for a specific cluster
aws eks describe-cluster --region eu-central-1 --name myapp-eks-cluster --query 'cluster.resourcesVpcConfig'

# Generate kubeconfig for kubectl access
aws eks update-kubeconfig --region eu-central-1 --name myapp-eks-cluster

# Test cluster connectivity
kubectl get svc
```

## Architecture

**Online Boutique** is composed of 11 microservices written in different
languages that talk to each other over gRPC.

[![Architecture of
microservices](/docs/img/architecture-diagram.png)](/docs/img/architecture-diagram.png)

Find **Protocol Buffers Descriptions** at the [`./protos` directory](/protos).

| Service                                              | Language      | Description                                                                                                                       |
| ---------------------------------------------------- | ------------- | --------------------------------------------------------------------------------------------------------------------------------- |
| [frontend](/src/frontend)                           | Go            | Exposes an HTTP server to serve the website. Does not require signup/login and generates session IDs for all users automatically. |
| [cartservice](/src/cartservice)                     | C#            | Stores the items in the user's shopping cart in Redis and retrieves it.                                                           |
| [productcatalogservice](/src/productcatalogservice) | Go            | Provides the list of products from a JSON file and ability to search products and get individual products.                        |
| [currencyservice](/src/currencyservice)             | Node.js       | Converts one money amount to another currency. Uses real values fetched from European Central Bank. It's the highest QPS service. |
| [paymentservice](/src/paymentservice)               | Node.js       | Charges the given credit card info (mock) with the given amount and returns a transaction ID.                                     |
| [shippingservice](/src/shippingservice)             | Go            | Gives shipping cost estimates based on the shopping cart. Ships items to the given address (mock)                                 |
| [emailservice](/src/emailservice)                   | Python        | Sends users an order confirmation email (mock).                                                                                   |
| [checkoutservice](/src/checkoutservice)             | Go            | Retrieves user cart, prepares order and orchestrates the payment, shipping and the email notification.                            |
| [recommendationservice](/src/recommendationservice) | Python        | Recommends other products based on what's given in the cart.                                                                      |
| [adservice](/src/adservice)                         | Java          | Provides text ads based on given context words.                                                                                   |
| [loadgenerator](/src/loadgenerator)                 | Python/Locust | Continuously sends requests imitating realistic user shopping flows to the frontend.                                              |
