## What is Terraform State

Terraform stores information about the infrastructure it manages, called Terraform state.

It is a map of real resources and metadata to your configuration. It keeps track of metadata to improve performance for large infrastructures.

## Why is Terraform state so important?

Terraform state stores bindings between objects in a remote system and resource instances declared in the configuration.

When Terraform creates a remote object in response to a change of configuration, it will record the identity of that remote object against a particular resource instance.

Now, this object is available to be potentially updated or deleted, in response to future configuration changes.

## What does the state file look like?

Any Terraform state file follows the json format.

Note: Terraform state files are not designed to be edited manually. **Manual editing can break the state and is not recommended.**

## How does one update the state file?

Terraform provides command line tools to:
- update the state
- remove resources
- move resources from one state file to another

It is also possible to manually import resources to the state.

## Where does Terraform save the state?

- By default, Terraform stores state in local files with the name terraform.tfstate.
- Terraform also creates a backup of the state named `terraform.tfstate.backup`

The state can also be stored remotely on 3rd party providers like AWS S3, GCS, azure and many others. Remote state storage is also ideal for team collaboration.

Tip: If you are using remote storage, a good practice will be to switch on versioning support to be able to revert or refer to previous states if needed.