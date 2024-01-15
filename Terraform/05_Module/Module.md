Modules are containers for multiple resources that are use together.
# Module Basics

Infrastructure-as-code tools allow easy code re-usability and sharing. 

In Terraform, modules are the main way to package and reuse resource configurations. Terraform modules are defined by any set of Terraform configuration files ( `.tf` or `.json` ) in a folder. These files contain multiple resources that are used together.

![[module.png]]
*file tree*

Modules provide abstractions for more complex platform change-sets. This is similar to functions in any programming language. 

Here are a few use-cases for modules: 
1. Create staging and test environments 
	- Modules can utilize the same code to provision multiple, different environments. 
	- It also allows to maintain consistency between environments. 
2. Create re-usable code 
	- Modules can hide complex rules and dependencies and expose only relevant information to your environment variables. 
3. Use 3rd party ready modules to speed up development process 
	- There are many open source modules available on GitHub that can do super complex tasks, like provisioning complex infrastructure like RDS creation https://github.com/terraform-awsmodules/terraform-aws-rds.

## Types of Modules

- **Root Module**: Default module containing all `.tf` files in main working directory
- **Child Module**: A separate external module referred to from a `.tf` file

### Module sources:

- Local paths: ``` source = "../web-app"```
- Terraform Registry: 
```
module "consul" {
	source = "hashicorp/consul/aws" 
	version = "0.1.0"
}
```
- GitHub
- Bitbucket
- generic Git, Mercurial repositories
- HTTP URLs
- S3 buckets
- GCS buckets
## How to use modules in your configuration? 

To include a module from your main configuration, we can use the following syntax: 

```
module "`<NAME>`" { 
	source = "<SOURCE>" 
	
	[CONFIG ...] 
}
```

- **source** defines the location of the module. It can be a local directory or a git repository as well. 
- **name** is the unique module identifier 
- **config** specify the different module options 

Let’s look at an example of a module definition: 

```
module "worker" { 
	source = "modules/terraform-aws-instance"

	# Input variables
	role              = "worker" 
	instance_type     = "m5d.large" 
	tags              = local.tags 
	subnet            = local.subnets 
	security_groups   = [module.security_groups.global, module.security_groups.worker]
	dns_private_zone  = module.dns.private_zone_id 
	dns_public_zone   = module.dns.public_zone_id 
	instances_count   = 3 root_size = 40 
}
```

## Module Inputs 

Module inputs, similar to arguments in functions, are defined with the variable block, specifying the name, type and default value. 

**info**: Variables without a default value are required and will throw an error if they’re not defined. 

As a common rule, variables should be defined in variables.tf file. 

These variables can be referred by using the following syntax: `var`. Let’s look at an example: 
```
resource "aws_instance" "instance" { 

	... 
	instance_type = var.instance_type 
	...
	
```

By re-using modules, we get the option to create 2 instance types: 
For example: 

```
module "worker" { 
	source           = "modules/terraform-aws-instance" 
	role             = "worker" instance_type = "m5d.large" 
	security_groups  = [module.security_groups.global, module.security_groups.worker] 
	dns_private_zone = module.dns.private_zone_id 
	dns_public_zone  = module.dns.public_zone_id 
	instances_count  = 3 root_size = 40 
} 

module "database" { 
	source           = "modules/terraform-aws-instance" 
	role             = "database" 
	instance_type    = "m5d.xlarge" 
	security_groups  = [module.security_groups.global, module.security_groups.database] 
	instances_count  = 2 
	root_size        = 120 
}
```

## Local Variables 

Local variables are similar to arguments/input variables and can be declared by using the locals block: 

For example: 

```
locals { 
	tags = { 
		Product = "Example" 
		Environment = var.environment 
		} 
	subnet_e = module.subnet-e.subnet_id 
	subnet_f = module.subnet-f.subnet_id 
	subnets = [local.subnet_e, local.subnet_f] 
	availability_zones = ["us-east-1e", "us-east-1f"] }
```

*Note*: locals should be declared in a separate file: locals.tf. 

### Referencing local variables 

Use the following syntax to use local variables: local. 

```
module "worker" { 
	source   = "modules/terraform-aws-instance" 
	role     = "worker" 
	tags     = local.tags 
	subnet   = local.subnets
}
```

**info**: Functions in local variables It is possible to use Terraform functions in locals to simplify syntax in resource configuration

## Module Outputs 

Module outputs or output variables are equivalent to the return values of functions in any programming language. 

All output variables must be defined in an `outputs.tf` file, by using the output block:

### Referencing output variables 

These variables can be accessed using the following syntax: 
```
module.<MODULE_NAME>.<OUTPUTNAME> 
```

For example: 
```
module "worker" { 
	source        = "modules/terraform-aws-instance" 
	instance_type = "m5d.large" 
	... 
}

resource "aws_route53_record" "teleport" { 
	zone_id   = module.dns.private_zone_id 
	name      = "workers" 
	type      = "A" 
	records   = module.worker.public_ips 
	ttl       = "30" 
}
```

## What Makes a Good Module?

- Raises the abstraction level from base resource types
- Groups resources in a logical fashion
- Exposes input variables to allow necessary customization + composition
- Provides useful defaults
- Return outputs to make further integration possible
## Versions 

### Why are versions important? 

Let’s say we have two environments, testing and production. 

Now, if we store our module as a directory, and both the environments are pointing to the same directory, any change in the directory will affect both environments on the very next deployment. 

This can interrupt the live, production environment while testing new configurations. This can be avoided by using versions for both the environments!

**What does that mean?** It means that we should use separate version controlled (git, mercurial, bitbucket, etc.) repositories for each or for all modules. 

We can reference a specific version by using tags. 

For example: 

```
module "webserver" { 
	source = "github.com/example/terraform-aws-webserver?ref=v0.0.1" 
}
```
Here, the ref parameter allows to define tag or branch to use for import.

*Pro tip* While testing, it is quicker sometimes to set the reference back to a local folder. This allows for faster iterations.