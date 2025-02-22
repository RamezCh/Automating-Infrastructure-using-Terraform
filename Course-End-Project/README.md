# Automating Infrastructure using Terraform

## Problem Statement

Use Terraform to build a virtual machine and install other required automation tools in it.

- Launch an EC2 instance using Terraform
- Connect to the Instance
- Install Jenkins, Java and Python in the instance

Tools required: Terraform, AWS acc with security credentials, Key-pair

## Solution

### Step 1: Check if Terraform Core is installed on the Lab VM:

```bash
terraform --version
```

![](./resources/project-1.png)

### Step 2: Create AWS user and Security credentials:

In the search box, give IAM and select it.

![](./resources/project-2.png)

You will be on IAM dashboard

On the left side -> Click on Users

![](./resources/project-3.png)

Click on Create user button

![](./resources/project-4.png)

under User deetails give User name as Terraform -> click on next

![](./resources/project-5.png)

Select 3rd option, Attach policies directly -> scroll down and from lsit search for AdministratorAccess and click on it

![](./resources/project-6.png)

Scroll down -> click next -> scroll down and click on Create User

![](./resources/project-7.png)
![](./resources/project-8.png)

Click on the user name Terraform

![](./resources/project-9.png)

Click on security credentials tab

![](./resources/project-10.png)

Scroll down to access key -> click on create access key

![](./resources/project-11.png)

Click on CLI -> Scroll down -> Select the box of "I understand..." -> click on next

![](./resources/project-12.png)

Click on create access key

![](./resources/project-13.png)

Click Show Secret Access Key -> Copy Access Key and save it on notepad for later -> Click Done

![](./resources/project-14.png)

### Step 4: Create a Key Pair in AWS, we will use this Key Pair to connect to the EC2  Instance

Select keypairs in the search box adn click on key pairs

![](./resources/project-15.png)

Click on create key pair

![](./resources/project-16.png)

Give name: demo1

Select Key Pair type: ED25519

Select Private key file format: .pem

Click Create Key Pair

![](./resources/project-17.png)

### Step 5: Get EC2 AMI Instance type t2 micro

In Search Box type EC2 -> In Services EC2 Look at Top features -> Click on Instances

![](./resources/project-19.png)

Click on Launch Instances

![](./resources/project-20.png)

Scroll Down -> Under Application and OS Images -> Click Amazon Linux

![](./resources/project-21.png)

Look for Amazon Machine Imagine (AMI) -> Press on Box under it -> Select Amazon Linux 2 AMI (HVM) -> Save AMI ID on notepad

![](./resources/project-22.png)

Scroll down -> Under Instance Type maek sure t2.micro is selected

![](./resources/project-23.png)

Scroll down -> Select demo1 for key pair(login)

![](./resources/project-24.png)

Scroll down -> Click on Launch Instance

![](./resources/project-25.png)

### Step 6: Prepare the terraform configuration file with provider and resources blocks

![](./resources/project-18.png)

In the configuration file we will use:
- Provider: aws
- Resource: security_group
- Resource: aws_instance
- Resource: aws_network_interface_sg_attachment

![](./resources/project-27.png)

![](./resources/project-26.png)

### Step 7: Execute the terraform configuration

Initialize Config -> Ensure correct -> create execution plan and review changes to be made -> apply the config and create the specified resources

```bash
terraform init
terraform validate
terraform plan
terraform apply --auto-aprove
```

![](./resources/project-28.png)

![](./resources/project-29.png)

![](./resources/project-30.png)

![](./resources/project-33.png)

### Step 8: Validate and check if the tools have been installed on the VM or not

Go to EC2 -> Instances

![](./resources/project-37.png)

Select Instance 1 -> Click Connect

![](./resources/project-34.png)

Scroll down -> Click connect

![](./resources/project-35.png)

```bash
sudo su -
java --version
python --verison
jenkins --version
```

![](./resources/project-36.png)
