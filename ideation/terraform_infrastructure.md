
# Terraform Infrastructure

```hcl
provider "aws" {
 region = "us-east-1"
}

resource "aws_vpc" "school_platform" {
 cidr_block = "10.0.0.0/16"
}

resource "aws_db_instance" "postgres" {
 engine = "postgres"
 instance_class = "db.t3.medium"
 allocated_storage = 20
}
```
