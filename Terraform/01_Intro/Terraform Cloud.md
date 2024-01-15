## When is it a good idea to use local state? 

State should be stored locally in situations where it is not required to be shared (for eg. working in teams)

## So is remote state only useful for team?

Remote state should be used not just when a team is working together, it also makes it possible to access the state from different locations! 

For example: 

- State can be stored in an S3 bucket with configured access (also useful to enable versioning in the bucket). But S3 bucket won’t resolve conflicts if several people run terraform apply at the same time. This might even lead to some resources being lost.

## Solution: state locking

Terraform can use state locking to prevent concurrent runs of Terraform against the same state

**important**: Locks should be supported by the backend of your application. Click [here](https://developer.hashicorp.com/terraform/language/settings/backends/remote) to find the list of backends which support locking state

# Terraform Cloud 

One other solution is to use Terraform Cloud. It can be used for free but they also have paid features for bigger teams. It is also possible to use a self-hosted version. 

Terraform Cloud allows more than just secure storage for your state with access control. 

You can connect VCS repository to the cloud to automate changes, deploy, and see the result of execution. 

Terraform Cloud has 2 execution modes:

- **remote** - Your plans and applies occur on Terraform Cloud’s infrastructure. You and your team have the ability to review and collaborate on runs within the app. 
- **local** - Your plans and applies occur on machines you control. Terraform Cloud is only used to store and synchronize state. 

You can share workspaces and configure access control to terraform cloud workspaces to specific people.