# Overview

Terraform lets us create a wide range of resources that work with multiple providers. 

## Creating resources with special requirements 

Let’s say we have to create resources that have very specific requirements, like: 
- Resources in the same cloud provider, but different regions 
- Multiple instances of the same resource, with different names 
- Resources with declared dependencies

Terraform’s meta-arguments let us do exactly these things and much more, making it super flexible to create any resource with special requirements! 
## What are meta-arguments? 

Meta-arguments are special, built-in resource attributes that can be specified in the resource configuration. These special attributes can be used with any type of resource. 

# `provider` meta-argument

We already know that a provider configuration is created using a provider block as follows:

```
provider "aws" { 
	region = "us-east-1" 
} 
```

Most arguments in a provider block are defined by the provider itself. 

Alias It is possible to define multiple configurations for the same provider in a single `.tf` file. 

Then, we can decide which provider to use per resource. We can do this by defining multiple provider blocks with the same name. 

For all but one of the provider blocks, we specify the alias meta-argument to provide an extra name attribute which helps us differentiate between each of them. 

*important*: A provider block without an alias argument is the default configuration for that provider. 

Let’s take a look at an example: 

We can define multiple configurations for our aws provider as follows:

```
# The default provider configuration; 
# resources that begin with “aws_” will use it by default 

provider “aws” { 
	region = “us-east-1” 
}
```

## Additional provider configuration for west coast region

```provider “aws” { alias = “west” region = “us-west-2” }```

#### The provider meta-argument

The `provider` meta-argument If we want any `resource` to use an alternate `provider`  configuration (one defined with `alias`), we can do so by using the `provider` meta-argument as an attribute in the `resource` definition. 

We can set its value by using the following syntax: 

provider = `<PROVIDER_NAME>, <ALIAS>`

Let’s look at an example: 

```
provider “aws” { region = “us-east-1” }
```

```
provider “aws” { 
	alias = “west” 
	region = “us-west-2” 
} 

resource “aws_instance” “web-servers” { 
	ami = “ami-abcd” 
	instance_type = “t3.micro” 
	provider = aws.west 
	tags = { 
		Name = “ExampleAppServerInstance” 
	} 
	
}
```

# `lifecycle` meta-argument 

Lifecycle allows us to specify resource behavior when it changes or needs to be updated..

It has a bunch of different options that lets us specify exactly how we want our resource to manage changes/updates to its attributes. 

They are: 
- `ignore_changes`: prevents Terraform from trying to revert metadata being set elsewhere
- `create_before_destroy`: can help with zero downtime deployments
- `prevent_destroy`: causes Terraform to reject any plan which would destroy this resource
- `replace_triggered_by`
## `ignore_changes` (list of resource attributes) 

The `ignore_changes` argument lets us list the attributes that we want our resource to ignore if it is changed/updated in the configuration. 

This can come in handy in cases where changes in attributes can be destructive. 

For example, we can set it to ignore the `ami` attribute to prevent resource recreation in case it’s changed. 
```
lifecycle { 
	ignore_changes = [ 
		ami, 
	]
} 
```

## `create_before_destroy` (boolean) 

Terraform’s default behavior is to destroy the old resource before creating a new one if an in-place, live resource update is not possible. This might require more time to restore functionality or cause a bigger downtime. 

We can modify this behavior by using the `create_before_destroy` argument and setting it to true which will instruct Terraform to create the new resource before destroying the old one. 

```
lifecycle { create_before_destroy = true } 
```
*info*: Usage of the **`create_before_destroy`** argument requires that the two resources must be able to co-exist, with no conflicts in names of IPs, amongst other things. 

## `prevent_destroy` (boolean) 

The **`prevent_destroy`** argument can be set to true in order to reject any plan with an error, if it would destroy a real infrastructure object associated with the respective resource. 

This prevents the resource from being destroyed by the terraform destroy command as well. 
### **`replace_triggered_by`** (list of resource or attribute references) 

 **`replace_triggered_by`** can trigger resource replacement if any of the referenced resources/attributes change.

## Force resource reprovisioning 

The existence of the `replace_triggered_by` option of the lifecycle meta-argument hints at the possibility that we might need to explicitly trigger resource replacement. 

This can be due to a multitude of reasons, like: 
- provider retiring the resource
- the resource needs to be recreated with new attributes that are marked to ignore changes 

In order to do that, Terraform has the `replace` option with the terraform plan and terraform apply commands. 

Doing this will trigger resource replacement regardless of attribute changes. 

*info*: terraform taint is deprecated and not recommended 
- Terraform also has an optional command called terraform taint `<resource>` to mark the resource for replacement. 
- Doing this will trigger replace whenever terraform apply is run next. To undo this, `terraform untaint` `<resource>` can be used. 
- These commands have been deprecated and not recommended anymore, but still good to know while they’re still available.

# `depends_on` Meta-arguments

Terraform automatically generates dependency graph based on references

if two resources depend on each other (but not each other data), `depends_on` specifies that dependency to enforce ordering. 

Is a way to explicit a dependency between multiple resources that have a implicitly dependency.

```
resource "aws_iam_role" "example" {
	name = "example"
	assume_role_policy = " ... "
}

resource "aws_iam_instance_profile" "example" {
	role = aws_iam_role.example. name
}

resource "aws_iam_role_policy" "example" {
	name = "example"
	role = aws_iam_role.example.name
	policy = jsonencode({
	"Statement" = [{
		"Action" = "s3 :* ",
		"Effect" = "Allow",
		}],
	})
}

# For example, if software on the instance needs access to S3, trying to create the aws_instance would fail if attempting to create it before the aws_iam_role_policy

resource "aws_instance" "example" {
	ami           = "ami-a1b2c3d4"
	instance_type = "t2.micro"
	
	iam_instance_profile = aws_iam_instance_profile.example
	depends_on = [
		aws_iam_role_policy.example,
	]
}
```
