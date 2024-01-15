Terraform is written in the Go programming language and doesn’t have any special dependencies.

It can be installed on any Linux, Windows or Mac machine and can be used in CI/CD to apply infrastructure changes automatically.

# Installing from binary

Terraform is distributed as a binary package and can be installed using the popular package managers (eg. Homebrew, Chocolatey, etc.).

To install Terraform on your own machine, you can follow the instructions on the Terraform website and download the relevant version - [Download Terraform](https://developer.hashicorp.com/terraform/downloads)

Here’s an example on how to download the Terraform binary on a Linux machine from the terminal:

```
wget -O- https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg

echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list

sudo apt update && sudo apt install terraform
```

Use official documentation [Install Terraform](https://developer.hashicorp.com/terraform/install)
# Installing using `tfenv`

`tfenv` is an open-source version management tool for Terraform. It can be used to:
- easily install any version of Terraform
- update Terraform from one version to another
- manage multiple environments with different versions of Terraform

You can check out the `tfenv` `Github` repo for more information on its usage.

You can check the installed version of Terraform by pasting the following
command in the terminal on the left:

	terraform --version

# Example Creation an AWS EC2 instance

In order to define an EC2 instance by AWS (the provider), only 2 parameters are required:
- *ami* - an AMI id for the instance
- *instance_type* - the size of the instance.

```
resource "aws_instance" "example" {
	ami = “ami-0ff8a91507f77f867” # us-west-1
	instance_type = “t2.micro”
}
```

The AMI ID used in this configuration:
- ami-0ff8a91507f77f867 is specific to the us-east-1 region.
- t2.micro - instance type with 1 vCPU and 1 GiB of memory
# Initializing

**First, we need to initialize the state and check all dependencies. This is done by running `terraform init` in the terminal.**

Running this command will also:
- check version compatibility between modules, providers and code.
- download all used plugins and modules to the .terraform module

**The .terraform module should usually be excluded from version control.**
Use terraform init or terraform init -upgrade to update all components to their latest versions.

# Locks

You should also be able to see the .terraform.lock.hcl file in the file tree. It locks the dependencies to the exact versions (hashes) for components in the configuration.

Terraform will always reselect versions of components it uses from the lock file unless -upgrade is specified for the init command.
If a particular provider has no existing recorded selection, then the latest version that matches the given version constraint will be selected and added in the lock file.