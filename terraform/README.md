## Create 3-tier architecture manually using terraform

### What is Terraform and how does it work

Terraform is an open-source infrastructure as code (IaC) software tool that provides a consistent CLI workflow to manage hundreds of cloud services.

A Terraform module is a set of Terraform configuration files in a single directory.

Terraform Cloud is an application that helps teams use Terraform together. 


### Installing terraform with brew
```ruby
brew tap hashicorp/tap
brew install hashicorp/tap/terraform
brew update
brew upgrade hashicorp/tap/terraform
terraform -help
```
### Enable tab completion
```ruby
touch ~/.bashrc
terraform -install-autocomplete
```

### Initialize the directory
```ruby
terraform validate
terraform fmt
terraform validate
```

Step 1: 

Step 2: 