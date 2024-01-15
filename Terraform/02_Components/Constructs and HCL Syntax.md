Refer to [Terraform Documentation](https://developer.hashicorp.com/terraform/language) for more information

# HCL: Overview

The **native, low-level syntax** of the Terraform language is called HCL (HashiCorp Configuration Language).

```
io_mode = “async”

service “http” “web_proxy” {
	listen_addr = “127.0.0.1:8080”
	process “main” {
		command = [“/usr/local/bin/awesome-app”, “server”]
	}
	process “mgmt” {
		command = [“/usr/local/bin/awesome-app”, “mgmt”]
	}
}
```


# Key Elements

HCL is built around two primary concepts:

- **attributes**: An attribute assigns a value to a particular name.
- **blocks**: Block is a container for content. Each block has a type. *Depending on what the type is, each block requires a certain number of labels to follow*. The body of each block is delimited by { and }. In this example, `resource "aws_instance" "example" { ... }` the block type is `resource` and expects 2 labels to follow:
	- “`aws_instance`” indicate the `aws` instance/service
	- “`example`” stand for the name given to the resource 
# Identifiers

Attribute/Argument names, block type names, and the names of most Terraform-specific constructs like resources, input variables, etc. are all called **identifiers**.

Identifiers can contain letters, digits, underscores (\_), and hyphens (-). 

*The first character of an identifier must not be a digit, to avoid ambiguity with literal numbers.*

# Comments

The Terraform language supports three syntaxes for comments:
- \# begins a single-line comment, ending at the end of the line.
-  // also begins a single-line comment, as an alternative to #.
- /* and \*/ are start and end delimiters for a comment that might span over multiple lines.

# Providers and Resources

## Providers

^e91596

We define providers in Terraform with the provider block.

```
provider "aws" { 
	region = "us-east-1" 
}
```

Providers are used to interact with Infrastructure as a service (IaaS) or Platform as a service (PaaS) APIs. 

In the configurations it must declare which providers they require so that Terraform can install and use them. Additionally, some providers require configuration (like endpoint URLs or cloud regions) before they can be used.

Providers also define which resources are available to **create**, **update** or **delete**.

## Resources

^da43eb

**A Terraform resource is any infrastructure component that is managed by providers.**

The resource type defines any subsequent resource attributes as well as the formatting for their values. 

```
resource “aws_instance” “foo” {
	ami = “ami-005e54dee72cc1d00” # us-west-2
	instance_type = “t2.micro”
	
	network_interface {
		network_interface_id = aws_network_interface.foo.id
		device_index = 0
	}
	
	credit_specification {
		cpu_credits = “unlimited”
	}
}
```

It's important to note that **any changes** in attributes either triggers a change in the resource or the recreation of the resource if an in-place change is not possible.
### How can we update an attribute without triggering changes?

see more information on [[Meta-Arguments]]

The `lifecycle` attribute allows to define behavior of a resource when an attribute changes.
Let's look at an example where we can allow a resource to ignore changes to certain attributes:

```
lifecycle {
ignore_changes = [
	ami,
	key_name,
	user_data,
	]
}
```

*In this example, if the ami, key_name or user_data attributes in the block don’t match, resource recreation or change won’t be triggered on terraform apply.*

# Input and Output Variables

## Input variables

You can define input variables to serve as Terraform configuration parameters. Input variables are defined by using a variable block, within a file named *variables.tf*

```
variable environment {
	default = “production”
	type = string
	description = “Environment type”
}

variable region {
	default = “us-east-1”
	type = string
	description = “Region for resources”
}

variable foo {
	type = number
	description
}
```

**Note**: *Adding a **default value** makes that variable **optional***

### What variable types are supported in Terraform?

- `string` - string values
- `number` - numeric value (fractional and whole)
- `bool` - boolean
- `list` - a sequence of values `["us-west--1a", "us-west-1c"]`
- `map` - a group of values identified by named labels `{name = "Mabel", age = 52}`
- `null` - a value that represents absence or omission

### Validation

- Type checking happens automatically
- Custom conditions can also be enforced
### How do we set or assign values to these variables?

*(In order of precedence // lowest -> highest)*

1. Manual entry during plan/apply
2. Default value in declaration block
3. TF_VAR_`<name>` environment variables
4. `terraform.tvars` file
5. `*.auto.tfvars` file
6. command line -var or -var-file

Typically variables can be assigned values in the following ways:
- Individually, with the `-var` command line option
	- We can set individual variables on the command line by using the `-var` option with the `terraform plan` and `terraform apply` commands
	```
	terraform apply -var="region=us-east-2"
	```
- In a Variable Definitions File (`.tfvars`)
	- If **multiple** variables have to be set, it's best to use a **variable definitions file** (a file with the `.tfvars` extension)
	- This file consists **only** of **variable name assignments** and follows the **same** basic syntax as any other Terraform language file.
	- This file can then be specified on the **command line** by using the `-var-file` option.
	Here's an example of a `.tfvars` file:
	```
	foo = 2
	environment = "staging"
	region = "us-west-1"
	```
	and this is how it can be specified on the command line:
```
	terraform apply -var-file="vars.tfvars"
```
- Using environment variables
	- As a fallback measure, Terraform will **always** search its **own process environment** for variables named in the following format: `TF_VAR_<variable_name>`
## Output variables

If we compare the Terraform language to other programming languages, output variables would be like the return values of a function.

Yes, it is possible to return values from Terraform execution. These can be hashed/encrypted passwords or server IPs.

Basically, these values expose infrastructure information available for other Terraform configs to use, as well as on the command line.

Output variables are defined by using the `output` block
```
output "ip" {
	value = "${aws_instance.foo.ip_address}"
}
```

# Referencing Variables

## How can we reference/use the defined variables?

Any defined variable can be used in the resources attributes using **var.<variable_name>**. 

```
resource "aws_instance" "foo" { 
	ami = var.ami instance_type = var.type 
}
```
## Interpolation

Terraform makes it possible to use interpolation to reference variables and other named values. This is done by using the following syntax - ${ }. This not only allows variable referencing, but also lets us work with more complex cases like conditions.

Let’s look at some instances of interpolation in this example:

```
resource “aws_instance” “foo” {
	ami = var.ami
	instance_type = var.type
	tags {
		“Name” = “foo-${var.environment}”
	}
	subnet_id = “${var.env ==”production" ? var.prod_subnet : var.dev_subnet}"
}
```

# Functions and Templates

## Functions

The Terraform language includes a number of built-in functions that you can call from within expressions to transform and combine values.
### What are expressions?

Expressions refer to or compute values within a configuration, and can range from simple literal values, like strings or numbers, to complex ones such as conditions, using built-in functions, etc.

Click [here](https://developer.hashicorp.com/terraform/language/expressions) to learn more about Expressions from the Terraform Docs.

The general syntax for function calls is: 
<FUNCTION_NAME>(<ARGUMENT_1>, ...)

For example:
``` 
max(5, 12, 9)
```

**important**: Terraform does not support user-defined functions. You can only use built-in functions in  Terraform

### What can we do with Terraform’s built-in functions?

Terraform’s built-in functions can be classified in the following groups:
- Numeric functions - to manipulate numbers
- String functions - to manipulate strings
- Collection functions - to manipulate arrays and other collections
- Encoding functions - support base64, json, csv, yaml encoding and decoding
- Filesystem functions - to manipulate filesystem paths
- Date and Time functions
- Hash and Crypto functions
- IP Network functions - to manipulate IP addresses
- Type Conversion functions

## Templates

Templates can be used to store large strings of data. The template provider
exposes the data sources for other Terraform resources or outputs to consume.

The data source can be a file or an inline template, for example:

```
data “template_file” “web” {
	template = “{file("{path.module}/ips.json”)}"
	vars {
		web_ip = "${aws_instance.foo.ip_address}"
	}
}
```