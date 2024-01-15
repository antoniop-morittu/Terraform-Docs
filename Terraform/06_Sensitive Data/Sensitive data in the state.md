Terraform state (whether it’s local or remote) can contain “sensitive” data. Depending on the organization structure and policies, the definition of sensitive can change but it raises an important question: 

## What kind of sensitive data can exist in the state? 

At the very minimum, if we have compute instances in our infrastructure, the state contains all the resource IDs as well as all resource attributes associated with them. 

If our infrastructure also has databases as resources, it would mean that the state might even contain initial passwords. 

### Local state storage 

When we use local state on our systems, the state is stored as plain-text in JSON files. 

### Remote state storage 

When we use remote state, the state is only held in system memory when it is used by Terraform. 

### How about encryption? 

The encryption of remote state depends on the type of remote storage. At the very least, it is important to have access control (ACL) rules for the state and encrypt it at-rest. 

**Note**: It is important to highlight the fact that everyone who has access to the state, will be able to see all sensitive information stored in it. There are **no** exceptions.

# Sensitive variables

## What if we wish to define variables with sensitive data? 

Such variables in the `variables.tf` file must be declared `senstive` by using the following attribute: 

```sensetive = true``` 

For example: 
```
variable "db_username" { 
	description = "Database administrator username" 
	type = string 
	sensitive = true 
} 

variable "db_password" { 
	description = "Database administrator password" 
	type = string 
	sensitive = true 
} 
```

Because we flagged the new variables as sensitive, Terraform redacts their values from its output when we run a plan, apply, or destroy command. 

**info**: The AWS provider considers the password argument for any database instance as sensitive, whether or not you declare the variable as sensitive, and will redact it as a sensitive value. 

You should still declare this variable as sensitive to make sure it’s redacted, especially if you reference it in locations other than the specific password argument.

**Pro Tip**: Do not store passwords as open text in any terraform config file like: 
```
# DO NOT DO THAT
resource "aws_db_instance" "database" { 
	allocated_storage = 5 
	engine = "mysql" 
	instance_class = "db.t2.micro" 
	username = "admin" 
	password = "secure_password"
}
```

### What to do instead? 

Let’s define a `secret.tfvars` file, with all sensitive variables assigned their values. For example: 

```
db_username = "admin" 
db_password = "secure_password" 
```

Now, we can pass this file when we run terraform apply like: 

```
terraform apply -var-file="secret.tfvars"
```
And, these variables can also be used in the `main.tf` configuration file like any other resource input variable. 

```
resource "aws_db_instance" "database" { 
	allocated_storage = 5 
	engine = "mysql" 
	instance_class = "db.t2.micro" 
	username = var.db_username 
	password = var.db_password 
} 
```

We can also use environment variables to pass sensitive variables to terraform: 

```
TF_VAR_db_password=secure_password
```

## Generating and storing passwords 

### Does declaring variables sensitive redact them from the state? 

No! Even if a variable is marked as sensitive, it is still stored in the state in open text. This is a big problem when it comes to passwords! 

**important**: Initial password change recommended If a password was generated when a resource is provisioned, it is recommended to change the initial password to a different one. 

Another way to avoid password leaks is to store passwords encrypted with asymmetric algorithms. An example of this is in the login profile for an AWS IAM users: 
```
resource "aws_iam_user" "john" { 
	name = "john" 
	path = "/users" 
	force_destroy = true 
} 

resource "aws_iam_user_login_profile" "login" { 
	lifecycle { 
		ignore_changes = [ 
			password_length, 
			password_reset_required, 
			pgp_key, 
		] 
	} 
	user = aws_iam_user.john.name 
	pgp_key = local.john_aws_pgp 
}
```

We use the optional `pgp_key` argument which allows to encrypt the generated password with a specified PGP public key. 

By doing this, the state will export an `encrypted_password` attribute, that will contain the base64 encoded encrypted password for the user. 

The password can be decrypted only with a specific private key for the user. 

Without providing `pgp_key`, the resource will generate a random password which will be recorded in the state as open text. In this case, you should consider using the `password_reset_required` argument which prompts users to change the password on their first login.
