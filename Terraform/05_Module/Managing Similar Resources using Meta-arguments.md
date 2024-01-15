
More info about [[Meta-Arguments]]

Meta-arguments are built-in resource attributes that can be specified in the resource configuration. They are used to create any type of resources with special requirements.

One of the most important use cases of meta-arguments is to deploy multiple instances of the same resource. This  is similar to the concept of loops in the programming language.

The following two meta-arguments are built specifically for this purpose:

- `count`
- `for_each`

## `count` and `for_each`

### The `count` meta-argument

The count meta-argument accepts a whole number and creates that many instances of the resource specified. 

**info**: Each created instance is associated with its own, unique infrastructure object.

Let’s look at an example:
```
resource "aws_instance" "example" { 
	count = 3 
	ami = ami_123456788 
	instance_type = "t2.micro" 
	} 
```

Now, let’s say we want to reference a specific instance of the resource we created in the above example with count. We do that using the following syntax: 

```
aws_instance.example[0] 
```

**info**: We can set the value for count using variables as well. 

```
count = var.instance_count
```

If we wish to reference count in assigning values to any other resource attributes, we can do so by using `count.index`, where index is the distinct index number for that instance. 

```
resource "aws_instance" "example" { 
	count = var.instance_count 
	ami = ami_123456788 
	instance_type = "t2.micro" 
	tags = { 
		Name = "example${count.index}" 
		} 
	}
```

## The `for_each` meta-argument

Like count, the for_each meta-argument creates multiple instances of a resource. However, instead of specifying the number of resources, the `for_each` meta-argument accepts a map or a set of strings. 

Let’s see what that means by looking at an example: 

```
locals { 
	instances = { 
		“webserver” = m5d.medium 
		“worker” = m5d.xlarge 
		“database” = m5d.xxlarge 
		} 
	} 
	
resource “aws_instance” “example” { 
	for_each = local.instances 
	ami = ami_123456788 
	instance_type = each.value 
	tags = { 
		Name = each.key 
		} 
	}
```

This example config creates 3 instances according to the map instances.

**Note**: keys should be “known values”, and can’t use variables or functions. sensitive values like passwords should not be used in values

## The Set Datatype in Terraform 

As we know, the `for_each` meta-argument not only accepts a map of `keyvalue` pairs, but also accepts a set of strings.

A set is a group of unique values without any secondary identifiers or ordering. In Terraform, sets are one of 3 available collection types - a type that allows the grouping of multiple values of another type into a single one. 

**info**: Other Collection Types The other two collection types in Terraform are list and map 

### How does type conversion work in Terraform? 

Terraform will automatically convert types wherever required. There will be rare cases where explicit type conversion is necessary (for eg. normalizing types returned as outputs, providing input to the `for_each` meta-argument, etc.) 

In order to do that, Terraform provides a function called `toset` 

Let’s look at an example: 

``` 
locals { 
	subnet_ids = toset([ “subnet-abcdef”, “subnet-012345”, ]) 
	}
	
resource “aws_instance” “server” { 
	for_each = local.subnet_ids
	ami = “ami-a1b2c3d4” 
	instance_type = “t2.micro” 
	subnet_id = each.key 
	tags = { 
		Name = “Server ${each.key}” 
		} 
	} 
```

As we can see in this example, we used `toset` to convert the **list** of **strings** into a set. 

We know that the `for_each` meta-argument provides an object called each with two attributes that can be referenced in expressions as - `each.key` and `each.value` 

When we pass a **map** to `for_each`, we can refer to the keys and values in each key-value pair by using these attributes. However, when we pass a set, `each.key` and `each.value` are the same.


## Dependencies

In most cases, Terraform manages dependencies automatically. The order of the definition of resources is not important. 

Terraform will try to create/remove resources in parallel if possible, but if a resource depends on another one, Terraform will execute the action one resource at a time. For example: 

```
resource “aws_instance” “example_a” { 
	ami = ami_123456788 
	instance_type = “t2.micro” 
}

resource “aws_eip” “ip” { 
	vpc = true 
	instance = aws_instance.example_a.id 
}
```

On running `terraform apply`, the terminal output will be something like this: 

```Terminal Output
aws_instance.example_a: Creating... 
aws_instance.example_a: Still creating... [10s elapsed] 
aws_instance.example_a: Still creating... [20s elapsed] 
aws_instance.example_a: Still creating... [30s elapsed] 
aws_instance.example_a: Creation complete after 31s [id=i04a809d45fa8c4cd0] 
aws_eip.ip: Creating... 
aws_eip.ip: Creation complete after 1s [id=eipalloc01b57879885d037fa] 
Apply complete! 
Resources: 2 added, 0 changed, 0 destroyed. 
```

As we can see in this example, Terraform will wait for the creation of the `aws_eip.ip` resource until it is done with creating `aws_instance.example_a`.

## Reprovision 

### How does reprovisioning resources work with dependencies?

Let’s look at the example from the previous page again: 

```
resource “aws_instance” “example_a” { 
	ami = ami_123456788 
	instance_type = “t2.micro”
 } 
 
 resource “aws_eip” “ip” { 
	 vpc = true 
	 instance = aws_instance.example_a.id
 }
```

If we decide to change or reprovision `aws_instance.example_a` by recreating it, AWS will try to retain all the dependencies and just reassign them to new instances. But if we taint `example_a` and run `terraform apply`, the terminal output will look something like this: 

```
Terraform will perform the following actions: 
# aws_eip.ip will be updated in-place 
~ resource "aws_eip" "ip" { 
	id = "eipalloc-01b57879885d037fa" 
	~ instance = "i-04a809d45fa8c4cd0" -> (known after apply) 
	tags = {} 
	# (12 unchanged attributes hidden) 
} 

# aws_instance.example_a is tainted, so must be replaced 
-/+ resource "aws_instance" "example_a" { 
	~ arn = "arn:aws:ec2:useast-1:948252217950:instance/i-04a809d45fa8c4cd0" -> (known after apply) 
	~ associate_public_ip_address = true -> (known after apply) 
	~ availability_zone = "us-east-1e" -> (known after apply) 
	.... 
Plan: 1 to add, 1 to change, 1 to destroy.
```

We can see that the dependency is not recreated (the IP will be moved to the new host).

## The `depends_on` meta-argument 

Even though Terraform automatically manages dependencies, there will be times when we need to define the dependency explicitly. 

This usually happens when a resource relies on another resource’s behavior, but does not access any of that resource’s data in its arguments. 

To do this, we can use the depends_on meta-argument. This will handle hidden resources that Terraform cannot automatically infer. 

**Note**: Try to avoid using `depends_on` and if needed, only use it in rare scenarios when an explicit dependency needs to be created. 

Things to consider when using `depends_on` 
- Always include a comment to explain why using `depends_on` is necessary. 
- `depends_on` causes Terraform to create a more conservative plan. The plan may modify more resources than necessary. 
- Adding explicit dependencies can increase the time it takes to build your infrastructure. Terraform must wait until the dependency object is created before continuing. 
- It is recommended to use expression references to create implicit dependencies whenever possible. 

Let’s look at some scenarios where `depends_on` can be used: 
- When your infrastructure isn’t aware of any dependencies related to the architecture of the application 
- Running an SQL query after the database is created 
- Fetching secrets only after vault storage is deployed.