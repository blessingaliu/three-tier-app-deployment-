## Terraform 3-Tier Architecture Example

This repository contains Terraform code to deploy a 3-tier architecture on AWS. It is designed to create and manage infrastructure for a scalable web application, including networking, database, and autoscaling resources.

### Table of Contents
- [Overview](#overview)
- [Architecture](#architecture)
- [Prerequisites](#prerequisites)
- [Getting Started](#getting-started)
  - [Clone the Repository](#clone-the-repository)
  - [Configure Variables](#configure-variables)
  - [Run Terraform Commands](#run-terraform-commands)
- [Modules](#modules)
- [Outputs](#outputs)
- [License](#license)

### Overview
This project demonstrates how to deploy a simple 3-tier application architecture using Terraform. The architecture includes:
- **Networking Layer**: Configures a VPC, subnets, and security groups.
- **Database Layer**: Provisions an RDS instance.
- **Application Layer**: Sets up an auto-scaling group with a load balancer.

### Architecture
1. **Networking**: Creates a VPC with public and private subnets, and configures security groups.
2. **Database**: Provisions an Amazon RDS instance for persistent data storage.
3. **Application**: Configures an auto-scaling group with EC2 instances, using a load balancer for traffic management.

## Prerequisites
To deploy this architecture, ensure you have the following:

1. [Terraform](https://www.terraform.io/downloads.html) installed (version >= 0.15).
2. An AWS account with appropriate permissions.
3. An SSH key pair for EC2 instances.
4. Terraform Cloud account (optional) for remote state storage.

## Getting Started

### Clone the Repository
```bash
git clone <repository-url>
cd <repository-folder>
```

### Configure Variables
Update the `terraform.tfvars` file with your specific values:
```hcl
namespace = "your-namespace"
region    = "your-aws-region"
ssh_keypair = "your-ssh-key-name"
```

### Run Terraform Commands
Initialize Terraform:
```bash
terraform init
```

Validate the configuration:
```bash
terraform validate
```

Plan the deployment:
```bash
terraform plan
```

Apply the configuration:
```bash
terraform apply
```

Destroy the infrastructure when no longer needed:
```bash
terraform destroy
```

## Modules
This repository uses the following modules:

1. **Networking**: Configures VPC, subnets, and security groups.
    - `source = "./modules/networking"`
2. **Database**: Provisions an RDS instance.
    - `source = "./modules/database"`
3. **Autoscaling**: Creates an auto-scaling group and attaches a load balancer.
    - `source = "./modules/autoscaling"`

## Outputs
After applying the configuration, Terraform will output important information, such as:
- `lb_dns_name`: The DNS name of the load balancer.

## License
This project is licensed under the MIT License. See the LICENSE file for details.

---

For any issues or contributions, feel free to open a pull request or submit an issue.
