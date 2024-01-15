# Overview

The most important element in the Terraform language, `resources`

Resources are how we specify real infrastructure objects that we want to create in our Terraform configuration. These objects can be anything from disks and compute instances to virtual
networks.

In order to understand the concept of resources, we need to focus on two things:
- Resource Declaration:
	- We declare resources in the Terraform syntax by using resource blocks.
- Resource Behavior
	- Once a resource is declared, Terraform handles that declaration in a certain way when the configurations are applied. 
	- This behavior lets us understand the inner workings of Terraform and how it manages the created resources.
## Resource Declaration

Here’s the resource block example:

```
resource "aws_instance" "server" { 
	ami = "ami-a1b2c3d4" 
	instance_type = "t2.micro" 
	}
```
We also know that in order to declare resource blocks, we need two labels:
- type
- local name

Basically, a resource block declares a specific type of resource (aws_instance ) with a specific local name ( server ).

## Resource Arguments

Resources can have both, mandatory and optional arguments or properties.

In order to make an argument mandatory, it has to be specified as such in the resource block.

### *info*: What if a property is not set?

If not specified, Terraform uses the default value or an empty value for any resource argument.
**Note**: Default values can change between different versions of providers, resulting in a resource update when we run terraform apply with existing resources.

### What if we decide to set the value for a resource argument in the future?

For example, if we provisioned an `aws_instance` with an empty `key_name` and later decided to set it to some value, Terraform will destroy the resource to create a new one with the updated `key_name`.

If we wish to avoid resource recreation in this case, we need to use the lifecycle meta-argument to ignore changes in the `key_name`.

## Resource Behavior

All the settings and properties of the real infrastructure objects that we wish to create, are specified in the Terraform configuration.

### *info*: What happens when we’re writing a new configuration?

In this case, given that it’s the first time, anything we define in this configuration will exist only in the configuration.

In order to create the real infrastructure objects they represent, we apply the Terraform configuration.

### How does Terraform apply a configuration?

Terraform follows this general behavior for all resources of any given type:
- **Resource Creation**: 
	- Terraform creates resources that don’t exist outside of the configuration file (have no bindings to any real infrastructure objects) in its state.
- **Resource Destruction**: 
	- If a resource managed by the state doesn’t exist in the configuration, it destroys the associated infrastructure objects.
- **Updates (in-place)**: 
	- If any arguments of an existing resource have changed, Terraform tries to update it in-place, without recreating it.
- **Resource reprovisioning**: 
	- If a resource with changed arguments cannot be updated in place, Terraform destroys and re-creates it.

## Ghost Resources

Let’s revisit an example:
```
resource "aws_instance" "server" { 
	ami = "ami-a1b2c3d4" 
	instance_type = "t2.micro"
	}
```
#### How many resources will be created with this config?

It feels obvious that only 1 resource would be created. 

However, the provider (AWS) automatically creates other (ghost) resources, not managed by Terraform, that are required for the instance we created. (e.g., an EBS drive and a network interface)


### *info*:  Is it possible for Terraform to manage those resources?

Yes. It is possible to adjust this behavior and create a network interface as a separate resource. This can then be attached to the instance using its `network_interface` argument.

By doing this, the network interface will also be directly managed by Terraform, which means we can update the network interface and tags without affecting the instance.

```
resource "aws_instance" "server" { 
	ami = "ami-a1b2c3d4" 
	instance_type = "t2.micro"
	
	network_interface {
	  ...
	}
  }
```


In the same way, we can recreate the instance without affecting the network interface. It will be attached to the new server instance, allowing to keep the same public and private IP address.

## Resource Attributes

All specified resource attributes can be accessed by using Expressions. This also makes it possible to use that information to configure other resources!

Here’s the syntax to use expressions for referencing resource attributes:

`<RESOURCE_TYPE>.<LOCAL_NAME>.<ATTRIBUTE>`

### Can resources have more attributes than the ones specified in the configuration?

Resources often expose read-only attributes (think of return values from a function of any programming language). This information comes from a remote API, and is usually things that can’t be known before resource creation.

Let’s explore this with the help of an example:

``` 
resource "aws_instance" "server" { 
	ami = "ami-a1b2c3d4" 
	instance_type = "t2.micro"
	}
```

We know that this config creates an AWS EC2 instance. Upon creation, here are some attributes that are exported and available for use:

- arn : ARN (Amazon Resource Name) is a unique identifier for this instance
- public_ip : the public IP address assigned to the instance
- instance_state

[Full List exported attributes for an EC2 instances](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/instance#attributes-reference)

### How can we use these exported attributes?

Let’s look at an example where we can use the exposed `public_ip` to create a public record for our server:

```
resource “aws_instance” “server” {
	ami = “ami-a1b2c3d4”
	instance_type = “t2.micro”
}

resource “aws_route53_record” “server” {
	zone_id = “public.zone_id”
	name = “www.example.com”
	type = “A”
	ttl = 300
	records = [aws_instance.server.public_ip]
}
```

*important*:  [AWS Route53 Records](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/route53_record) are used for domain registration, instance health checks and DNS routing

Take a look at the records attribute in the aws_route53_record resource. Using the `public_ip` allows to dynamically manage our resource, keeping it in sync with the original instance by creating a dependency.

# Data Sources

It’s not the best idea to have static or hard-coded values for all attributes of a resource

The perfect example to understand this better is to look at the AMI attribute.

## What is an AMI ?

AMI (Amazon Machine Image) is like a template configured with an OS, an application server and other software, used for the creation of virtual servers (EC2 instances).

AMI types are categorized according to region, amongst other factors. This means that it is possible to hardcode an AMI id of an Ubuntu image like ami-06878d265978313ca for the us-east-1 region.

### Can we use the same AMI id for different zones/regions?

If we try to apply the exact same Terraform configuration except the provider region, all the associated AMI ids will be different!  In such a situation, if we hard code this attribute, we’ll have to change it manually every time!

In order to avoid hardcoding resource attributes, we can use Data sources! Data sources allow Terraform to use information from outside the current configuration: whether that means it’s defined by a different configuration, or completely external! They can also help us to provision resources from latest versions.

We can access a data source by using a special data resource, which can be
specified by using the data block. Let’s look at an example:

```
data “aws_ami” “ubuntu” {
	most_recent = true
	owners = [“099720109477”]
	filter {
		name = “name”
		values = ["ubuntu/images/ubuntu---amd64-server-*"]
	}
}

resource “aws_instance” “server” {
	ami = data.aws_ami.ubuntu.id
	instance_type = “t2.micro”
	lifecycle {
	ignore_changes = [
		ami,
		]
	}
}
```

Here, the data "`aws_ami`" resource will find the most recent AMI image according to the filter.
This means that every time we run `plan` or `apply` and a new version of Ubuntu is available, Terraform will try to recreate it.

### Why are data sources important?

Data sources are not only useful to find 3rd party resources but also allow splitting our infrastructure into smaller, logical parts, making it more manageable.

Instead of referencing return/output variables, we can use data source search to find resources that are managed manually or by a different Terraform project.