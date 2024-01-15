
## How to Update resources

To make changes to our real infrastructure, we can update the respective resource attributes in the `.tf` configuration file. 

Once we’re done making changes, we can run the terraform plan command to see exactly what changes Terraform is going to make to our infrastructure to make sure it aligns with the specified configuration. 

### What happens if we update unique identifiers or other critical resource attributes? 

Certain changes can be destructive if the specified attributes are incompatible with the  current/assigned values. This means that every resource will have certain attributes that will force resource recreation, whereas it might be possible to update some attributes in-place. 

Let’s take a look at an example: 

```
resource "aws_instance" "web-servers" { 
	ami = "ami-abcd" 
	instance_type = "t3.micro" 
	count = 3 
	tags = { 
		Name = "ExampleAppServerInstance"
		} 
	}
```

Now, if we decide to change the `ami` of the EC2 instance, it will lead to recreation of the resource. But tags and `instance_type` can be updated IN place.
### In-place updates

^2dcf48

**IN places updates**: *There is a good chance that the resource might be stopped and restarted to perform in-place update operations.*

Once we change the attributes we wish to update in our `.tf` config file, we can run `terraform plan`. It highlights the property which is going to change with the ~ symbol.

In this case, running terraform plan should print something like this on your terminal: 
```# aws_instance.server will be updated in-place 
~ resource “aws_instance” “server” { 
	id = “i-0d074f3fc6cc64953” 
	~ instance_type = “m5.large” -> “t3.nano” 
	tags = {} 
# (29 unchanged attributes hidden) 
# (7 unchanged blocks hidden) }
```

This update won't cause instance recreation. The property will be updated on the *live* resource by stopping and restarting it with the new `instance_type`. 
#### How do we know if an attribute change will recreate the resource? 

Let's say we decide to update the `ami` of our instance. Now, if we run `terraform plan`, we will see an output that looks something like this: 
```
# aws_instance.server must be replaced 
-/+ resource "aws_instance" "server" { 
	~ ami = "ami0ea1c7db66fee3098" -> "ami-0228d2cc2850106ef" 
	# forces replacement 
	~ arn = "arn:aws:ec2:useast-1:948252217950:instance/i-0d074f3fc6cc64953" -> (known after apply) 
```
As we can see, the attribute forcing resource recreation will be highlighted with forces replacement.
