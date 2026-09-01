# Terraform AWS Cloud Security Learning Project

Terraform is basically a tool that allows IaC which stands for Infrastructure as Code. It allows us to define and manage cloud infrastructure using code rather than the graphical user interface.

So basically instead of logging in to the AWS cloud website and then clicking buttons to create servers and databases, networks, security groups, IAM roles, buckets and all that other cloud bullshit, you write a configuration file describing what you want and then Terraform builds it for you.

Now I know what you are thinking.

I thought the same.

Why the hell must I learn yet another damn language?

Why can't I just click buttons?

Seems easier no?

It all comes down to saving time.

Saving time building.

Saving time fixing.

Saving time auditing.

How you ask?

Good Question Dumbass.

## Why Terraform?

### Version Control

Ever heard about Git?

It basically tracks shit.

Every change is reviewed, documented, and easily reversible.

You make a change and don't like it?

Roll right back.

And since you typed the infrastructure yourself, you also know what exactly you did, even a month from now.

But with a GUI, nobody is tracking those clicks of yours.

So if something breaks, have fun opening tabs.

### Instant Replica

If I, as your boss, cause let's face it I am, ordered you to build me a cloud environment, what's in it I leave to your imagination.

But imagine it being painfully complex.

Tons of EC2s.

S3 buckets.

IAM roles.

Policies.

Networks.

Security groups.

Blah blah blah.

You spend a bunch of days building it.

But since you are a script kiddie, or in my language, a bitch, you used the GUI.

But since you are a good egg, you actually built it.

And then came running to me.

"Daddy daddy look I did it."

And just like your actual father who left you when you were an infant, I don't show any appreciation.

Instead I tell you to build exactly the same thing for the Staging environment as you did for the Production environment.

Now you have to do all the clicks again.

Maybe you don't remember what you did the first time.

Maybe you accidentally configure something differently.

Maybe you spend another few days doing the same shit.

But if you had written it using Terraform, you just have to change a variable or two and run the code again.

And in a few minutes, boom.

Another environment.

### Disaster Recovery

If your infrastructure gets deleted or compromised, your Terraform code acts as a blueprint to rebuild the entire system.

Rebuilding everything manually through a GUI relies on memory, screenshots, random documentation or some outdated wiki page nobody has touched since 2022.

Terraform gives you the infrastructure definition in code.

### Error-Free Scale

Deploying 50 identical servers through code takes roughly the same effort as deploying one.

Manual GUI clicking is slow and guarantees that eventually you are going to forget something.

Maybe you forgot a security group rule.

Maybe you forgot to enable something.

Maybe server number 40 has a completely different configuration from server number 1 because you were half asleep.

Code gives you consistency.

## There Are Other Tools

There are others like Terraform but since this is one of the most widely used ones, you are just learning this one.

The best place to begin learning it is the creators themselves, HashiCorp.

https://developer.hashicorp.com/terraform/tutorials

Now you will see Azure, AWS and other options in the beginning.

Just pick one.

Don't try both.

I know you think you're a smart ass but for once listen to PAPA.

I picked AWS.

## Learn The Basics First

Go through the Terraform course.

Here you will learn what Terraform is and how the basic Terraform files work.

You will come across files such as:

```text
terraform.tf
main.tf
variables.tf
outputs.tf
terraform.tfvars
```

You will learn what these files do, how Terraform manages infrastructure and how Terraform keeps track of the infrastructure it created.

They will also introduce you to HCP Terraform so you can manage Terraform projects, securely store state and collaborate with other people.

Take a few days and actually understand what is happening.

Don't just copy the commands and pray.

Also get a free AWS account because I know you're broke.

And one important thing.

When you are finished experimenting:

```bash
terraform destroy
```

Use that command.

AWS will happily keep charging you while you sleep.

# The Project

So after doing the above I went ahead and built one of my own.

If you don't know how to build this yet, read through it and try figuring out what each block is doing.

Don't immediately copy everything.

Try to understand what each resource is actually creating.

The project creates:

- A VPC
- A public subnet
- A private subnet
- An Internet Gateway
- A public route table
- A route table association
- A security group
- An S3 bucket
- S3 public access blocking
- An IAM role
- An IAM permissions policy
- An EC2 instance profile
- An Amazon Linux EC2 instance
- Terraform outputs for important resource information

The architecture is intentionally simple.

The point is not to build some massive production environment.

The point is to understand how all these pieces connect.

---

# main.tf

## Terraform Provider

This tells Terraform which provider we are using.

In this case we are using AWS.

```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}
```

## AWS Provider

This tells Terraform to use AWS and gets the region from our variable.

```hcl
provider "aws" {
  region = var.aws_region
}
```

---

# NETWORKING: VPC, subnets, internet gateway, routing

## VPC

A VPC is basically our own isolated network inside AWS.

Think of it as the main network that everything else is going to live inside.

```hcl
resource "aws_vpc" "main" {
  cidr_block           = var.vpc_cidr
  enable_dns_support   = true
  enable_dns_hostnames = true

  tags = {
    Name = "learning-vpc"
  }
}
```

The CIDR block defines the IP address range available inside the VPC.

The default value we use is:

```text
10.0.0.0/16
```

## Public Subnet

This creates the public subnet.

The important part here is:

```hcl
map_public_ip_on_launch = true
```

That means instances launched into this subnet automatically receive a public IP.

```hcl
resource "aws_subnet" "public" {
  vpc_id                  = aws_vpc.main.id
  cidr_block               = var.public_subnet_cidr
  availability_zone        = var.availability_zone
  map_public_ip_on_launch  = true

  tags = {
    Name = "learning-public-subnet"
  }
}
```

## Private Subnet

This creates the private subnet.

Notice there is no `map_public_ip_on_launch`.

Also later you will notice that we don't give this subnet a route to the Internet Gateway.

```hcl
resource "aws_subnet" "private" {
  vpc_id            = aws_vpc.main.id
  cidr_block        = var.private_subnet_cidr
  availability_zone = var.availability_zone

  tags = {
    Name = "learning-private-subnet"
  }
}
```

The idea is that the private subnet is actually isolated.

We aren't putting anything in it yet.

## Internet Gateway

The Internet Gateway is what allows the VPC to communicate with the internet.

Without this, having a public IP alone isn't enough to magically make your instance internet accessible.

```hcl
resource "aws_internet_gateway" "main" {
  vpc_id = aws_vpc.main.id

  tags = {
    Name = "learning-igw"
  }
}
```

## Public Route Table

Now we need to tell AWS where traffic should go.

This route says:

`0.0.0.0/0`

Basically everything that isn't part of the local network should go through the Internet Gateway.

```hcl
resource "aws_route_table" "public" {
  vpc_id = aws_vpc.main.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.main.id
  }

  tags = {
    Name = "learning-public-rt"
  }
}
```

## Route Table Association

Creating a route table doesn't automatically attach it to the subnet.

We need to associate the public subnet with the public route table.

This is what actually makes the subnet public.

```hcl
resource "aws_route_table_association" "public" {
  subnet_id      = aws_subnet.public.id
  route_table_id = aws_route_table.public.id
}
```

The private subnet does not have this public route.

There is also no NAT Gateway in this simple version.

So the private subnet is intentionally isolated.

---

# SECURITY GROUP for the EC2 instance

A Security Group acts like a firewall for our EC2 instance.

Here we are allowing SSH access only from the IP address we specify in `var.my_ip`.

```hcl
resource "aws_security_group" "ec2_sg" {
  name        = "learning-ec2-sg"
  description = "Allow SSH from my IP only, and all outbound"
  vpc_id      = aws_vpc.main.id

  ingress {
    description = "SSH from my IP"
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = [var.my_ip]
  }

  egress {
    description = "Allow all outbound"
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "learning-ec2-sg"
  }
}
```

This is important from a security perspective.

We aren't doing this:

```hcl
cidr_blocks = ["0.0.0.0/0"]
```

for SSH.

That would allow SSH attempts from anywhere on the internet.

Instead we use our own IP.

```text
YOUR_IP/32
```

The `/32` basically means one specific IP address.

---

# S3 BUCKET

Now we create an S3 bucket.

The bucket name comes from a variable because S3 bucket names have to be globally unique across AWS.

```hcl
resource "aws_s3_bucket" "data" {
  bucket = var.bucket_name

  tags = {
    Name = "learning-bucket"
  }
}
```

## S3 Public Access Block

Now this is an important one from a cloud security perspective.

We explicitly block public access.

```hcl
resource "aws_s3_bucket_public_access_block" "data" {
  bucket = aws_s3_bucket.data.id

  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}
```

This is the secure default we want.

Later, when you start looking at real-world cloud misconfigurations, you will see how dangerous it can be when settings like these are missing or incorrectly configured.

---

# IAM: role + policy

Now we get to IAM.

The idea here is least privilege.

Our EC2 instance should be able to read from the S3 bucket.

It should not automatically get access to everything in AWS.

## IAM Role

First we create the IAM role.

The role itself doesn't define what the EC2 instance can do.

It defines who is allowed to assume the role.

```hcl
resource "aws_iam_role" "ec2_s3_read" {
  name = "learning-ec2-s3-read-role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Action = "sts:AssumeRole"
        Effect = "Allow"
        Principal = {
          Service = "ec2.amazonaws.com"
        }
      }
    ]
  })
}
```

The important part is:

```text
ec2.amazonaws.com
```

We are saying that EC2 is allowed to assume this role.

## IAM Permissions Policy

Now we define what the role can actually do.

This role can:

```text
s3:GetObject
s3:ListBucket
```

That's it.

```hcl
resource "aws_iam_role_policy" "s3_read" {
  name = "learning-s3-read-policy"
  role = aws_iam_role.ec2_s3_read.id

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect = "Allow"
        Action = [
          "s3:GetObject",
          "s3:ListBucket"
        ]
        Resource = [
          aws_s3_bucket.data.arn,
          "${aws_s3_bucket.data.arn}/*"
        ]
      }
    ]
  })
}
```

This is the least privilege idea in practice.

We don't give the EC2 instance:

```text
AdministratorAccess
```

because that would be fucking stupid.

We give it exactly what it needs.

## Instance Profile

Now there is one slightly confusing part.

You can't directly attach an IAM role to an EC2 instance.

The EC2 instance uses an instance profile.

The instance profile contains the IAM role.

```hcl
resource "aws_iam_instance_profile" "ec2_profile" {
  name = "learning-ec2-instance-profile"
  role = aws_iam_role.ec2_s3_read.name
}
```

So basically:

```text
EC2
 |
 v
Instance Profile
 |
 v
IAM Role
 |
 v
IAM Policy
 |
 v
S3 permissions
```

---

# EC2 INSTANCE

Now we actually create the EC2 instance.

Before creating it, we need an AMI.

Instead of manually entering an AMI ID, we ask AWS for the latest Amazon Linux 2023 AMI.

```hcl
data "aws_ami" "amazon_linux" {
  most_recent = true
  owners      = ["amazon"]

  filter {
    name   = "name"
    values = ["al2023-ami-*-x86_64"]
  }
}
```

Now we create the instance.

```hcl
resource "aws_instance" "web" {
  ami                    = data.aws_ami.amazon_linux.id
  instance_type          = var.instance_type
  subnet_id              = aws_subnet.public.id
  vpc_security_group_ids = [aws_security_group.ec2_sg.id]
  iam_instance_profile   = aws_iam_instance_profile.ec2_profile.name

  tags = {
    Name = "learning-ec2"
  }
}
```

Notice how everything is connected.

The EC2 instance gets:

- The Amazon Linux AMI
- The instance type from a variable
- The public subnet
- The security group
- The IAM instance profile

And because the subnet is public and has a route to the Internet Gateway, the EC2 instance can communicate with the internet.

---

# variables.tf

Now let's look at the variables.

Variables are basically a way of making your Terraform configuration reusable.

Instead of hardcoding everything directly inside `main.tf`, we can define values separately and change them without rewriting our infrastructure.

## AWS Region

```hcl
variable "aws_region" {
  description = "AWS region to deploy into"
  type        = string
  default     = "us-east-1"
}
```

## VPC CIDR

```hcl
variable "vpc_cidr" {
  description = "CIDR block for the VPC"
  type        = string
  default     = "10.0.0.0/16"
}
```

## Public Subnet CIDR

```hcl
variable "public_subnet_cidr" {
  description = "CIDR block for the public subnet"
  type        = string
  default     = "10.0.1.0/24"
}
```

## Private Subnet CIDR

```hcl
variable "private_subnet_cidr" {
  description = "CIDR block for the private subnet"
  type        = string
  default     = "10.0.2.0/24"
}
```

## Availability Zone

```hcl
variable "availability_zone" {
  description = "AZ to place subnets in (keeping it simple with one AZ for this learning project)"
  type        = string
  default     = "us-east-1a"
}
```

We're keeping everything in one Availability Zone because this is a learning project.

A real production environment would normally be designed with availability and redundancy in mind.

## EC2 Instance Type

```hcl
variable "instance_type" {
  description = "EC2 instance type"
  type        = string
  default     = "t2.micro"
}
```

We're using `t2.micro` because it can be eligible for the AWS Free Tier depending on your account and current AWS terms.

Always check the current AWS pricing before deploying.

## S3 Bucket Name

```hcl
variable "bucket_name" {
  description = "Globally unique S3 bucket name -- CHANGE THIS before applying, bucket names must be unique across all of AWS"
  type        = string
  default     = "my-cloud-security-learning-bucket-change-me-12345"
}
```

Change this before running Terraform.

S3 bucket names must be globally unique.

So don't expect my example bucket name to work if somebody else already took it.

## Your IP Address

This one is important.

```hcl
variable "my_ip" {
  description = "Your public IP in CIDR form (e.g. 1.2.3.4/32) allowed to SSH into the instance"
  type        = string
}
```

There is intentionally no default.

You should provide your own public IP.

For example:

```text
1.2.3.4/32
```

The reason we don't put a default here is security.

We don't want to accidentally deploy an EC2 instance with SSH open to the entire internet.

---

# outputs.tf

Outputs are basically Terraform's way of saying:

"Hey, here is some information about what I just created."

Instead of digging through AWS to find IDs and addresses, Terraform can give them to us after deployment.

## VPC ID

```hcl
output "vpc_id" {
  description = "ID of the VPC"
  value       = aws_vpc.main.id
}
```

## Public Subnet ID

```hcl
output "public_subnet_id" {
  value = aws_subnet.public.id
}
```

## Private Subnet ID

```hcl
output "private_subnet_id" {
  value = aws_subnet.private.id
}
```

## EC2 Public IP

```hcl
output "ec2_public_ip" {
  description = "Public IP of the EC2 instance -- use this to SSH in"
  value       = aws_instance.web.public_ip
}
```

This is particularly useful because after Terraform creates the EC2 instance, you can get its public IP without going into the AWS console.

## S3 Bucket Name

```hcl
output "s3_bucket_name" {
  value = aws_s3_bucket.data.bucket
}
```

## IAM Role ARN

```hcl
output "iam_role_arn" {
  description = "ARN of the role attached to the EC2 instance"
  value       = aws_iam_role.ec2_s3_read.arn
}
```

---

# How To Deploy

Once you have your files:

```text
main.tf
variables.tf
outputs.tf
```

you need to initialize Terraform.

## 1. Initialize Terraform

```bash
terraform init
```

This downloads the AWS provider and prepares the directory.

## 2. Format Your Code

```bash
terraform fmt
```

This formats your Terraform files.

## 3. Validate The Configuration

```bash
terraform validate
```

If everything is correct, Terraform should tell you the configuration is valid.

## 4. Create terraform.tfvars

Create a file called:

```text
terraform.tfvars
```

Then put your values inside.

For example:

```hcl
aws_region        = "us-east-1"
vpc_cidr          = "10.0.0.0/16"
public_subnet_cidr = "10.0.1.0/24"
private_subnet_cidr = "10.0.2.0/24"
availability_zone = "us-east-1a"
instance_type     = "t2.micro"
bucket_name       = "your-globally-unique-bucket-name"
my_ip             = "YOUR.PUBLIC.IP/32"
```

Do not blindly copy my IP example.

Find your own public IP and put it there.

## 5. Plan

Before Terraform actually changes AWS, run:

```bash
terraform plan
```

This shows you what Terraform intends to create.

Read it.

Don't just hit apply like a maniac.

## 6. Apply

If everything looks correct:

```bash
terraform apply
```

Terraform will ask for confirmation.

Type:

```text
yes
```

and Terraform will build the infrastructure.

## 7. Check The Outputs

After deployment:

```bash
terraform output
```

You should see information such as:

```text
vpc_id
public_subnet_id
private_subnet_id
ec2_public_ip
s3_bucket_name
iam_role_arn
```

## 8. Destroy Everything

When you're done:

```bash
terraform destroy
```

Again.

Do not forget this.

AWS does not care that you are learning.

AWS will happily charge your ass while you sleep.

---

# What You Should Try To Understand

Don't just copy this project.

Try to understand the relationships.

Start with:

```text
VPC
 |
 +-- Public Subnet
 |     |
 |     +-- Route Table
 |     |     |
 |     |     +-- Internet Gateway
 |     |
 |     +-- EC2
 |           |
 |           +-- Security Group
 |           |
 |           +-- Instance Profile
 |                   |
 |                   +-- IAM Role
 |                         |
 |                         +-- IAM Policy
 |                               |
 |                               +-- S3
 |
 +-- Private Subnet
```

If you understand that diagram, you are already starting to understand how the pieces of AWS infrastructure fit together.

And this is where the security part becomes interesting.

Because once you understand how to build the infrastructure correctly, you can start deliberately breaking it.

For example:

What happens if the S3 public access block is missing?

What happens if SSH is allowed from `0.0.0.0/0`?

What happens if an EC2 instance gets `AdministratorAccess`?

What happens if the private subnet accidentally gets a route to the internet?

What happens if IAM permissions are much broader than they need to be?

Those are the kinds of things you should start looking for when you move from simply learning Terraform into learning Cloud Security and Infrastructure as Code security.

The goal isn't just to know how to deploy cloud infrastructure.

The goal is to look at Terraform code and immediately start asking:

"What the fuck is wrong with this configuration?"