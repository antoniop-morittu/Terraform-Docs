
One Config --> Multiple Environments

Development - Staging - Production

Two main approaches:
- Workspace: Multiple named sections within a single backend

```
-> ~ terraform workspace list
	default
	dev
	production
	staging
```

- File Structure: Directory layout providers separation, modules provide reuse

```
structure tree

|__ modules
|   |_ module-1
|   |   |_ main.tf
|   |   |_ variables.tf
|   |_ module-2
|       |_main.tf
|       |_variables.tf
|
|__ dev
|    |_ main.tf
|    |_ terraform. tfvars
|__ production
|    |_ main.tf
|    |_ terraform. tfvars
|__ staging
     |_ main.tf
     |_ terraform. tfvars
```

# Terraform Workspaces

Pros:
- Easy to get started
- Convenient `terraform.workspace` expression
- Minimizes Code Duplication

Cons:
- Prone to human error, easy to forget where put the changes
- State stored within same backend
- Codebase does not unambiguously show deployment configurations
# File Structure

Pros:
- Isolation of backends
	- Improved security
	- Decreased potential for human error
	- Codebase fully represents deployed state
Cons:
- Multiple `terraform apply` required to provision environments 
- More code duplication, *but can be minimized with modules*

## Environments + Components

- Further separation (at logical component groups) useful for large projects
	- Isolate things that change frequently from those which don't
- Referencing resources across configurations is possible using `terraform_remote_state`

# Terragrunt 

Tool by gruntwork.io that provides utilities to make certain Terraform use cases easier

- Keeping Terraform code DRY
- Executing commands across multiple TF configs
- Working with multiple cloud accounts

