
# Plan Changes with terraform plan

## Initialize

It start with creating a main.tf file with all the resources that are needed.

Initialize Terraform, means there are all required plugins and providers downloaded.

```
terraform init
```

	- It create the initial files and download the provider's dependecies

## Plan

With `terraform plan` compare the Terraform config (Desired State) in the with the desired state. (Actual State) 

In order to see what changes to the infrastructure we need to make to match the config file, we start by running the terraform plan command. 

```
terraform plan
```

	- On running this, Terraform reports what resources it is going to change/update or delete.

# Applying changes with Terraform 


```
terraform apply
```

When the terraform apply command is run.
- It executes the terraform plan command first
- Then, it asks for user input (yes/no) to confirm the planned changes, for example:

```
Do you want to perform these actions?
Terraform will perform the actions described above.
Only ‘yes’ will be accepted to approve.
```

## What happens if we run `terraform apply` again?

- in the terminal output, it begins by refreshing and querying the state of the resources it manages.
- It didn't find any changes in the configuration file (which makes sense) and it reports that no changes are needed!
- It ends the apply command by printing the following line in the output:
```
Apply complete! Resources: 0 added, 0 changed, 0 destroyed.
```

# List and Explore resources

## List Resources

We can browse the current state by using the terraform state list command.
```
terraform state list
```

this command returns a list of all the resources currently managed by Terraform.

## The count meta-argument

Each resource block is responsible for configuring only 1 real infrastructure object.

### What if we need multiple instances of the same resource?

Terraform has a count meta-argument which provides a solution to this by:
- Accepting a whole number as its value and creating that many instances of the respective resource.
- Each of these created instances has an independent and distinct infrastructure object associated with it.
- If we use count, we will see an array of resources with [n] as a postfix in the name.

## Explore Resources

The `terraform state list` command doesn’t give a detailed description of all the resources.

In order to get detailed information about a resource in the state, we use 
```
terraform state show <resource_name>
```

This command returns all the information about the created resource, including all default values!

## Other actions with the state

We know that Terraform state files are not designed to be edited manually.

Being an IaaS tool, Terraform provides a host of commands that can be run on the terminal to perform actions on the state.

They are:
1. `terraform state rm` This command disconnects cloud resources and removes them from Terraform’s state without destroying them. You can safely remove the respective resource blocks from your .tf file after running this command.
2. `terraform state pull` Pulls the latest Terraform state (if it is stored remotely) and prints it on the terminal as JSON
3. `terraform state push` Pushes local Terraform state to remote storage
4. `terraform state mv` This command is used to refactor Terraform files when resources have to be moved from one state to another.
5. `terraform import` This command is used to import existing cloud resources into Terraform control. This command is used to:
	- migrate from manually provisioned infrastructure
	- migrate between different systems.
## Terminate resources

To ensure we don’t exhaust free tier resources and run the risk of an AWS bill, let’s destroy the resource we created. 

This can be done by using a simple command: terraform destroy This will destroy all resources which are currently defined in the state

```
terraform destroy
```