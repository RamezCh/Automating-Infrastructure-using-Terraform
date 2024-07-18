# Automating Infrastructure using Terraform

## Description

## Project Agenda:
Use Terraform to provision infrastructure.

## Importance:
Infrastructure automation is critical for disaster recovery, testing, and development. Your organization is adopting the DevOps methodology and needs to set up a centralized server for Jenkins using Terraform.

## Key Concepts:
- Terraform: A tool to provision various infrastructure components.
- Ansible: A platform for managing configurations and deploying applications.

## Tools Required:
- Terraform
- AWS account with security credentials
- Keypair

## Solution:

### Step 1: Create User in AWS & Generate Access Keys
1. Log in to AWS Console:

- Open AWS Console.
- Search for and select IAM (Identity and Access Management).

2. Create a New User:

- On the left sidebar, click on Users.
- Click on Add User.
- Enter a username (e.g., terraform).
- Click on Next.
- Select the Attach policies directly option.
- Scroll down and select AdministratorAccess.
- Click Next and then Create User.

3. Generate Access Keys:

- Click on the username terraform.
- Go to the Security credentials tab.
- Scroll down to Access keys and click Create access key.
- Select Command Line Interface (CLI).
- Check the box to acknowledge the warning and click Next.
- Click Create access key and save the keys securely.

### Step 2: Configure AWS CLI

1. Open Terminal (Command Line Interface):

```bash
sudo su -
```

2. Configure AWS CLI:

```bash
aws configure
```
- Enter the access key and secret access key when prompted.
- Press Enter for region and output format options (not required).
```bash
cat ~/.aws/credentials
```

3. Create a Project Directory:

```bash
mkdir terraformProject
cd terraformProject
vim aws-infra.tf
```

### Step 3: Write Terraform Configuration

1. Configure AWS Provider and VPC:

```bash
provider "aws" {
  region = "us-east-1"
  shared_credentials_files = ["~/.aws/credentials"]
}

resource "aws_vpc" "sl-vpc" {
  cidr_block = "10.0.0.0/16"
  tags = {
    Name = "sl-vpc"
  }
}

resource "aws_subnet" "subnet-1" {
  vpc_id = aws_vpc.sl-vpc.id
  cidr_block = "10.0.1.0/24"
  depends_on = [aws_vpc.sl-vpc]
  map_public_ip_on_launch = true
  tags = {
    Name = "sl-subnet"
  }
}

resource "aws_route_table" "sl-route-table" {
  vpc_id = aws_vpc.sl-vpc.id
  tags = {
    Name = "sl-route-table"
  }
}

resource "aws_route_table_association" "a" {
  subnet_id = aws_subnet.subnet-1.id
  route_table_id = aws_route_table.sl-route-table.id
}
```

2. Initialize Terraform and Apply Configuration:

```bash
terraform init
terraform apply --auto-approve
```

### Step 4: Add Internet Gateway and Security Groups

1. Update Terraform Configuration:

```bash
resource "aws_internet_gateway" "gw" {
  vpc_id = aws_vpc.sl-vpc.id
  depends_on = [aws_vpc.sl-vpc]
  tags = {
    Name = "sl-gw"
  }
}

resource "aws_route" "sl-route" {
  route_table_id = aws_route_table.sl-route-table.id
  destination_cidr_block = "0.0.0.0/0"
  gateway_id = aws_internet_gateway.gw.id
}

resource "aws_security_group" "sl-sg" {
  name = "allow_tls"
  description = "Allow TLS inbound traffic and all outbound traffic"
  vpc_id = aws_vpc.sl-vpc.id

  ingress {
    from_port = 22
    to_port = 22
    protocol = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port = 8080
    to_port = 8080
    protocol = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port = 0
    to_port = 0
    protocol = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

resource "tls_private_key" "web-key1" {
  algorithm = "RSA"
}

resource "aws_key_pair" "web-key1" {
  key_name = "web-key1"
  public_key = tls_private_key.web-key1.public_key_openssh
}

resource "local_file" "web-key1" {
  content = tls_private_key.web-key1.private_key_pem
  filename = "web-key1.pem"
}

resource "aws_instance" "myec2" {
  ami = "ami-0195204d5dce06d99"
  instance_type = "t2.micro"
  key_name = "web-key1"
  subnet_id = aws_subnet.subnet-1.id
  security_groups = [aws_security_group.sl-sg.id]
  tags = {
    Name = "terraform-instance"
  }

  provisioner "remote-exec" {
    connection {
      type = "ssh"
      user = "ec2-user"
      private_key = tls_private_key.web-key1.private_key_pem
      host = self.public_ip
    }
    inline = [
      "sudo amazon-linux-extras install java-openjdk11 -y",
      "sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo",
      "sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key",
      "sudo yum install jenkins -y",
      "sudo systemctl start jenkins",
      "sudo systemctl enable jenkins"
    ]
  }
}
```

2. Apply Updated Configuration:
3. 
```bash
terraform init
terraform apply --auto-approve
```

### Step 5: Connect to the Instance and Verify Jenkins Installation

1. Navigate to the AWS Console:

- Open the AWS Console.
- Go to the EC2 Dashboard.
- Find your instance from the list (look for the instance with the "terraform-instance" tag).

2. Connect to the Instance:

- Select your instance and click on the "Connect" button.
- You will see several connection options. Choose "Connect using EC2 Instance Connect".
- Click Connect (Basically default options)

3. Once connected to your EC2 instance, check if Jenkins is installed:
```bash
sudo su -
java --version
python --version
jenkins --version
```
[Sample End Results](Endscreen.png)
