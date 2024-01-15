
1. Providers (AWS, GC, AZURE) ^6fcefd

Terraform interacts with many cloud platforms and services using their API. This is possible by Terraform plugins, which are called *providers*.

*Providers* are specified in Terraform configuration code, which specify the services it needs to interact with.

Once specified, each *provider* shows all the different resources and data types available for use.

Each provider has its own documentation, describing its resource types and their arguments. The [Terraform Registry](https://registry.terraform.io/browse/providers) includes documentation for a wide range of providers developed by HashiCorp, third-party vendors, and our Terraform community. Use the "Documentation" link in a provider's header to browse its documentation.
## How to use Providers

To use resources from a given provider, you need to include some information about it in your configuration. See the following pages for details:

 - [Provider Requirements](https://developer.hashicorp.com/terraform/language/providers/requirements) documents how to declare providers so Terraform can install them.
 - [Provider Configuration](https://developer.hashicorp.com/terraform/language/providers/configuration) documents how to configure settings for providers.    
 - [Dependency Lock File](https://developer.hashicorp.com/terraform/language/files/dependency-lock) documents an additional HCL file that can be included with a configuration, which tells Terraform to always use a specific set of provider versions.

 Construct and HCL Syntax in [[Constructs and HCL Syntax#^e91596|Providers]]

2. Resources ^377a38

*Resources* are defined as block of code, called *resource blocks*. 

````
resource "<PROVIDER>_<RESOURCE_TYPE>" "<NAME>" {
	...
}
````


Each resource block define one or more infrastructure objects. *Resources can be composed of objects from different providers.*

 More info about HCL Syntax in [[Constructs and HCL Syntax#^da43eb|Resources]] and how to provide them [[Provisioning Resources|Provisioning Resources]] and [[Updating Resources]]


3. Terraform State ^d7bf48

Terraform records and logs information about all the resources it creates and manages in a file, called the state file.

This state file acts as a source of truth for Terraform, and maps the objects in its configuration to the real world (remote instances and infrastructure).

## Track your infrastructure

Terraform keeps track of your real infrastructure in a state file, which acts as a source of truth for your environment. Terraform uses the state file to determine the changes to make to your infrastructure so that it will match your configuration.

It uses this state file and determines all the changes required to align infrastructure with the declared configuration.

 More info in [[Terraform State]]


