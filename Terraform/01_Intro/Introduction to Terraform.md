# What is Terraform

Is an open-source, infrastructure as code (iaC).

As an IaC tool:

- Define, change and manage data center infrastructure on multiple cloud platforms
- human-readable configuration language
- commit configurations to version control to track and reuse
- securely collaborate in a stable env.

It uses a declarative configuration language, allows users to describe the desired state of the cloud resources.

````
resource "aws_instance" "web-servers" { 
	ami = "ami-abcd" 
	instance_type = "t3.micro" 
	count = 3 
}
````

More in about Constructs and HCL Syntax in [[Constructs and HCL Syntax]]
# Why do we need Terraform?

Terraform provides clarity with respect to infrastructure and cloud
configuration. Amongst other things, it lets users:

- **spin** up the same configuration for testing and staging environments.
- **synchronize** changes making sure that testing environments are similar to production.
- **track** changes throughout deployments for review processes.

Terraform also makes **infrastructure knowledge transfer** very easy, and
improves security by limiting access for manual actions.

# Infrastructure

To deploy infrastructure with Terraform:

- **Scope** - Identify the infrastructure for your project.
- **Author** - Write the configuration for your infrastructure.
- **Initialize** - Install the plugins Terraform needs to manage the infrastructure.
- **Plan** - Preview the changes Terraform will make to match your configuration.
- **Apply** - Make the planned changes

# Configuration

The main purpose of the Terraform language is declaring [resources](https://developer.hashicorp.com/terraform/language/resources), which represent infrastructure objects. All other language features exist only to make the definition of resources more flexible and convenient.

A _Terraform configuration_ is a complete document in the Terraform language that tells Terraform how to manage a given collection of infrastructure. A configuration can consist of multiple files and directories.

The syntax of the Terraform language consists of only a few basic elements:

```
resource "aws_vpc" "main" {
  cidr_block = var.base_cidr_block
}

<BLOCK TYPE> "<BLOCK LABEL>" "<BLOCK LABEL>" {
  # Block body
  <IDENTIFIER> = <EXPRESSION> # Argument
}
```

- _Blocks_ are containers for other content and usually represent the configuration of some kind of object, like a resource. Blocks have a _block type,_ can have zero or more _labels,_ and have a _body_ that contains any number of arguments and nested blocks. Most of Terraform's features are controlled by top-level blocks in a configuration file.
- _Arguments_ assign a value to a name. They appear within blocks.
- _Expressions_ represent a value, either literally or by referencing and combining other values. They appear as values for arguments, or within other expressions.

The Terraform language is declarative, describing an intended goal rather than the steps to reach that goal. The ordering of blocks and the files they are organized into are generally not significant; Terraform only considers implicit and explicit relationships between resources when determining an order of operations.
# Module

Terraform's Module is the best way to package and reuse configuration

More info in [[Module]]

---

Components:
- [[Providers, Resources and Terraform State#^6fcefd|Providers]]
- [[Providers, Resources and Terraform State#^377a38|Resources]]
- [[Providers, Resources and Terraform State#^d7bf48|Terraform State]]

Setting Up Terraform
- [[Install Terraform]]
- [[Setting up IAM credentials for AWS authentication]]
- [[Steps to use Terraform]]