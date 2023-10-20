# Deployment-of-Infrastructure-Using-Terraform
Terraform configuration to automate the deployment of Google Cloud infrastructure.

## Overview
Terraform is an open-source tool designed to facilitate the secure and reliable management of infrastructure. It accomplishes this by translating APIs into declarative configuration files, which can be shared, treated as code, edited collaboratively, reviewed, and version-controlled.

In the context of a lab exercise, the goal is to craft a Terraform configuration that utilizes a module for the automated provisioning of Google Cloud infrastructure. This configuration aims to create a self-managed network, configure a firewall rule, and deploy two virtual machine instances within the specified environment.

## Objectives
- Create a configuration for an auto mode network
- Create a configuration for a firewall rule
- Create a module for VM instances
- Create and deploy a configuration

## Configure and Run
To configure and apply the execution plan, run the following commands:
```
terraform fmt
```

```
terraform init
```

```
terraform plan
```

```
terraform apply
```

## Conclusion
During this lab, we constructed a Terraform configuration utilizing a module to automate the deployment of Google Cloud infrastructure. With Terraform, we have the capability to generate progressive execution plans as our configuration evolves, enabling us to construct our overall infrastructure incrementally.

The "instance" module was a key element of this process, enabling us to efficiently reuse the same resource configuration for multiple instances while customizing properties through input variables. This modularity and flexibility are fundamental features of Terraform, enhancing our ability to maintain and scale our infrastructure provisioning.
