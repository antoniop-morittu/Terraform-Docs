
In order to execute Terraform commands to manage AWS resources, you first need:
- AWS account
- Associated Credentials that allow you to create and manage resources.

## How to Generate access keys

### Instructions 

- Log in to your AWS account as a Root User , and look for Security Credentials in the drop-down menu that shows up on clicking your username in the top right.
- Scroll down to Access Keys and you should see a button that says Create new key / Create access key
- You should now see the auto-generated Access key and Secret key
### Warning about creating access keys as a Root user

You might see a warning at this step that says Root user access keys are not recommended.
#### Why did we log in as a Root user in the first place?

Logging in as an IAM user (which is the part of the best practices) requires setting up proper permissions and access roles which is outside of the scope of this course.

In an organizational setting, you will always log in to your AWS IAM account where the organization’s admin (root) user will have configured all the necessary permissions required.

**Downloading the keys**: *You might want to download the access keys to your local system. An option to download the keys as JSON is available during key creation.*

## How to use AWS credentials in Terraform

[Create a variable set](https://developer.hashicorp.com/terraform/tutorials/cloud-get-started/cloud-create-variable-set)

To use the IAM credentials (Access key and Secret key) to authenticate the Terraform AWS provider:

- Create two Environment Variables
	- To set the Access key: 
		- Enter AWS_ACCESS_KEY_ID in the key field, and paste the generated Access key as its value
	- To set the Secret key: 
		- Enter AWS_SECRET_ACCESS_KEY in the key field, and paste the generated Secret key as its value.

or  in .tf file:

```
provider "aws" {
	region = "us-east-1"
	access_key = "<ACCESS-KEY>"
	secret_key = "<SECRET-KEY>"
}
```



