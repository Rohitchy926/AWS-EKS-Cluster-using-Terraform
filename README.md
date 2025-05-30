## Step 1: Launch an EC2 instance 
## Step 2 : Install terraform on that by using commands 

 # (i)Install HashiCorp GPG key and repo:

```
sudo apt-get update && sudo apt-get install -y gnupg software-properties-common
wget -O- https://apt.releases.hashicorp.com/gpg | gpg --dearmor | sudo tee /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
```

# (ii)Install Terraform:

```
sudo apt update
sudo apt install terraform
```

# (iii) check the installed version by - 
```
terraform -v
```

## Step 3 : Install ```AWS CLI```, ```Kubectl```, ```EKSCTL```:
 ```AWS CLI``` - AWS CLI is a unified tool for managing AWS services from the command line.

```
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"

apt install unzip

unzip awscliv2.zip
```

- ```Kubectl``` - kubectl is the command-line tool used to interact with Kubernetes clusters.
```
curl -O https://s3.us-west-2.amazonaws.com/amazon-eks/1.30.2/2024-07-12/bin/linux/amd64/kubectl

chmod +x kubectl

mv kubectl /usr/local/bin/kubectl
```    
- ```EKSCTL``` - eksctl is a command-line tool for managing Amazon EKS (Elastic Kubernetes Service) clusters.
```
curl -LO "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_Linux_amd64.tar.gz"

tar -xzf eksctl_Linux_amd64.tar.gz

sudo mv eksctl /usr/local/bin/

eksctl version
```


## Step 4 :  Creating a separate directory for Terraform 

 - (i) : Create Your Terraform Project Directory
```
mkdir eks-cluster-terraform
cd eks-cluster-terraform
```

- (ii) : Create Terraform Files- main.tf


```
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
  required_version = ">= 1.33.0"
}

provider "aws" {
  region = "us-east-1"
}

```


## Step 5: Create EKS Cluster using terraform

-(EKS Cluster & Node Group using AWS module) : eks.tf

```
module "eks" {
  source          = "terraform-aws-modules/eks/aws"
  version         = "20.8.4"

  cluster_name    = "eks-demo"
  cluster_version = "1.29"

  subnet_ids      = module.vpc.private_subnets
  vpc_id          = module.vpc.vpc_id

  enable_irsa     = true

  node_groups = {
    eks_nodes = {
      desired_capacity = 2
      max_capacity     = 3
      min_capacity     = 1

      instance_types = ["t2.medium"]

      key_name = "your-ec2-keypair-name"  # Replace or remove if not needed

      additional_tags = {
        Name = "eks-node"
      }
    }
  }

  tags = {
    Environment = "dev"
    Terraform   = "true"
  }
}

```

## Step 6:  (i) Create a VPC (optional: use EKS module with VPC support)

- vpc.tf

```
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "5.1.0"

  name = "eks-vpc"
  cidr = "10.0.0.0/16"

  azs             = ["us-east-1a", "us-east-1b", "us-east-1c"]
  public_subnets  = ["10.0.1.0/24", "10.0.2.0/24", "10.0.3.0/24"]
  private_subnets = ["10.0.11.0/24", "10.0.12.0/24", "10.0.13.0/24"]

  enable_nat_gateway = true
  single_nat_gateway = true

  tags = {
    Terraform = "true"
    Environment = "dev"
  }
}

```
- (ii) output.tf

```
output "cluster_name" {
  value = module.eks.cluster_name
}

output "kubeconfig" {
  description = "Kubeconfig to access the EKS cluster"
  value       = module.eks.kubeconfig
  sensitive   = true
}

output "cluster_endpoint" {
  value = module.eks.cluster_endpoint
}
```


## Step 7 - Creating Terraform resource now

```
terraform init
```

## Step 8-  Review & Apply

```
terraform plan
```

## step 9-Apply the changes:

```
terraform apply
```

## Step 10- Configure kubectl
- After Terraform finishes, extract the kubeconfig:
```
terraform output -raw kubeconfig > kubeconfig.yaml
export KUBECONFIG=./kubeconfig.yaml
```





