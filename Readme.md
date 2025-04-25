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
