# Bootstrapping Kubernetes (EKS) with Argo CD using Terraspace

This repository (see my gitlab repository [terraform-eks-argocd](git@gitlab.com:assessment4622826/terraform-eks-argocd.git)
 contains a Terraspace project that deploys the necessary AWS resources to bootstrap ArgoCD on an EKS cluster. The resources deployed include:

- VPC with public and private subnets
- IAM roles and instance profile
- NAT and Internet Gateway
- EKS cluster
- Namespace in Kubernetes for Argo CD

## Requirements

- Configured AWS profile with permissions to create the required resources

- Terraform and Terraspace  (Code was tested on version 1.5.1)

- AWS, Helm, and Kubernetes terraform providers

## Project Structure

```
.
├── iac
│   ├── app
│   │   ├── modules
│   │   └── stacks
│   │       └── resources
│   │           ├── argocd.tf
│   │           ├── cluster-roles.tf
│   │           ├── eks.tf
│   │           ├── nat_gw.tf
│   │           ├── output.tf
│   │           ├── security-group.tf
│   │           ├── vars.tf
│   │           └── vpc.tf
│   ├── config
│   │   ├── app.rb
│   │   └── terraform
│   │       ├── backend.tf
│   │       ├── provider.tf
│   │       └── vars.tf
│   ├── Gemfile
│   ├── Gemfile.lock
│   ├── README.md
│   └── Terrafile
└── README.md
```

## Remote Backend State Configuration

To ensure state is stored remotely and locked to prevent concurrent modifications, a remote backend and state locking using Amazon S3 and DynamoDB is configured. This is optimal when working in a team.

Before running terraform init, create an S3 bucket and DynamoDB table.
NB: The bucket name you choose should be globally unique, update the  `backend.tf` in the config/terraform

Create S3 Bucket for State Backend
``` aws s3api create-bucket --bucket <bucket name> --region <region> ```
If you are creating in a location outside of us-east-1, include this ````--create-bucket-configuration LocationConstraint= <region not us-east-1> ```

Create DynamoDB table for State Locking
``` aws dynamodb create-table --table-name <table name> --attribute-definitions AttributeName=LockID,AttributeType=S --key-schema AttributeName=LockID,KeyType=HASH --provisioned-throughput ReadCapacityUnits=1,WriteCapacityUnits=1 ```


## Provision Infrastructure

Clone the repository:

```
git clone https://github.com/your-repo/iac.git
```

Deploy the infrastructure:

```
cd iac
terraspace up resources
```

Cluster unreachable?
![alt text](img/Screenshot 2024-07-06 181026.png)

Ensure that the eks cluster and node group are done creating. This might take a while.
Check the console to confirm with the desired state.
You can connect to your cluster by updating your local kube config with the following command: 
```aws eks --region <aws-region> update-kubeconfig --name <cluster-name>```

Rerun the deploy

### Connect To ArgoCD

Copy the argocd_server_load_balancer in the generated output to your webbrowser. 
You can obtain the password for the application by running :

``` kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo ```

![alt text](img/Screenshot 2024-07-06 182501.png)

Destroy the infrastructure
`` terraspace down resources``
