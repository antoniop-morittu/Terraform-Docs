# Local Backend

By default Terraform start in local, that means the main configuration are stored in the local system.

Pros:
- Simple to get started

Contro:
- Sensitive values in plain text
- Uncollaborative
- Manual

# Remote Backend

The State file is stored in the cloud, could be:
- Terraform Cloud
- AWS S3
- Google cloud
- (Other cloud)

Pros:
- Sensitive data encrypted
- Collaboration possible
- Automation Possible

Contro:
- Increase complexity