# Lab: Create an EC2 using Terraform Modules

### Task 1: Create the Project Files
```
mkdir terraform-ec2-lab && cd terraform-ec2-lab
```
```
touch main.tf variables.tf outputs.tf
```

### Task 2: Define main.tf (EC2 using Terraform AWS Module)
```
nano main.tf
```
Paste the following code in **main.tf**
```
provider "aws" {
  region = var.aws_region
}

# -------------------------------
# EC2 INSTANCE MODULE
# -------------------------------
module "ec2_instance" {
  source  = "terraform-aws-modules/ec2-instance/aws"
  version = "5.0.0"

  name          = "yournam-ec2-instance"
  ami           = var.ami_id
  instance_type = var.instance_type
  vpc_security_group_ids = [var.security_group_id]
  # Optional: Use default VPC & subnet
  subnet_id = var.subnet_id

  tags = {
    Environment = "dev"
  }
}
```
To save the file Press **CTRL + O** Click Enter and Press **CTRL + X**

### Task 3: Define variables.tf (Input Variables)
```
nano variables.tf
```
Paste the following code in **variables.tf**
```
variable "aws_region" {
  description = "AWS region"
  type        = string
  default     = "us-east-1"
}

variable "ami_id" {
  description = "Amazon Machine Image ID for EC2"
  type        = string
  default     = "ami-084568db4383264d4" # Example for Ubuntu, change as needed
}

variable "instance_type" {
  description = "EC2 instance type"
  type        = string
  default     = "t2.medium"
}

variable "subnet_id" {
  description = "Subnet ID where the EC2 will be deployed"
  type        = string
  default     = ""  # Provide a subnet ID or leave it empty for default VPC
}

variable "security_group_id" {
  description = "Security Group ID for EC2"
  type        = string
  default     = ""  # Provide a security group ID 
}
```
To save the file Press **CTRL + O** Click Enter and Press **CTRL + X**

### Task 4: Define outputs.tf (Get EC2 Details)
```
nano outputs.tf
```
Paste the following code in **outputs.tf**
```
output "ec2_public_ip" {
  description = "Public IP of the EC2 instance"
  value       = module.ec2_instance.public_ip
}

output "ec2_id" {
  description = "EC2 Instance ID"
  value       = module.ec2_instance.id
}
```
To save the file Press **CTRL + O** Click Enter and Press **CTRL + X**

### Taks 5: Deploy the EC2 Instance
Run the following commands:
```
terraform init       
```
```
terraform plan
```
```
terraform apply -auto-approve
```

### Lab 6: Get EC2 Details
```
terraform output
```
This will show:
1. Public IP (Use to SSH into EC2)
2. EC2 Instance ID

### Lab 7: Destroy the EC2 Instance 
```
terraform destroy -auto-approve
```
