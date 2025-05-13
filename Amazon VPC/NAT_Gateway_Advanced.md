# Setting Up a NAT Gateway for Private Subnets in AWS

## Introduction

Today I learned how to set up a NAT Gateway for private subnets in AWS. This is a crucial component when working with VPCs that have both public and private subnets.

### Why NAT Gateway?

Resources in a private subnet cannot download packages or updates directly from the internet. A NAT Gateway solves this issue by allowing EC2 instances in private subnets to communicate with the internet to download packages and updates.

Important to note: NAT Gateway only provides one-way communication - resources in the private subnet can reach out to the internet, but external users cannot initiate connections to these resources through the NAT Gateway.

## Architecture Overview

Here's the setup I'm working with:
- VPC with public and private subnets
- Public subnet with Internet Gateway for internet access
- Private subnet connected to the public subnet via NAT Gateway
- EC2 instances in both subnets

## Step-by-Step Implementation

### 1. Create a VPC

1. Go to AWS Console and search for VPC
2. Click "Create VPC"
3. Select "VPC Only"
4. Set the following details:
   - Name: `test-vpc`
   - CIDR range: `12.0.0.0/16`
   - Keep default tenancy
5. Click "Create VPC"

### 2. Create Subnets

1. In the VPC dashboard, click on "Subnets" in the left navigation menu
2. Click "Create subnet"
3. Select the VPC created in the previous step
4. Create Public Subnet:
   - Name: `test-subnet-public-1a`
   - Availability Zone: eu-central-1a
   - CIDR: `12.0.1.0/24`
5. Add second subnet (Private):
   - Name: `test-subnet-private-1a`
   - Availability Zone: eu-central-1a
   - CIDR: `12.0.2.0/24`
6. Click "Create subnet"

### 3. Create Internet Gateway

1. In the VPC dashboard, click on "Internet Gateways" in the left navigation menu
2. Click "Create internet gateway"
3. Name it: `test-igw`
4. Click "Create internet gateway"
5. Attach it to the VPC:
   - Click "Actions" -> "Attach to VPC"
   - Select the VPC created earlier
   - Click "Attach internet gateway"

### 4. Create Route Tables

#### Public Route Table

1. In the VPC dashboard, click on "Route Tables" in the left navigation menu
2. Click "Create route table"
3. Set details:
   - Name: `test-rt-public`
   - VPC: Select the VPC created earlier
4. Click "Create route table"
5. Add the Internet Gateway route:
   - Select the created route table
   - Click "Edit routes"
   - Click "Add route"
   - Destination: `0.0.0.0/0`
   - Target: Select the Internet Gateway created earlier
   - Click "Save changes"
6. Associate with the public subnet:
   - Click "Subnet associations"
   - Click "Edit subnet associations"
   - Select the public subnet
   - Click "Save associations"

#### Private Route Table

1. Click "Create route table"
2. Set details:
   - Name: `test-rt-private`
   - VPC: Select the VPC created earlier
3. Click "Create route table"
4. Associate with the private subnet:
   - Click "Subnet associations"
   - Click "Edit subnet associations"
   - Select the private subnet
   - Click "Save associations"

### 5. Create NAT Gateway

1. In the VPC dashboard, click on "NAT Gateways" in the left navigation menu
2. Click "Create NAT Gateway"
3. Set details:
   - Name: `test-nat-gateway`
   - Subnet: Select the **public** subnet (important: NAT Gateway must be in a public subnet)
   - Connectivity type: Public
4. Allocate Elastic IP:
   - Click "Allocate Elastic IP" button
   - Select the newly allocated Elastic IP
5. Click "Create NAT Gateway" and wait for it to become available

### 6. Update Private Route Table with NAT Gateway

1. Go back to Route Tables
2. Select the private route table
3. Click "Edit routes"
4. Click "Add route"
5. Destination: `0.0.0.0/0`
6. Target: Select the NAT Gateway created earlier
7. Click "Save changes"

### 7. Create EC2 Instances

#### Public EC2 Instance

1. Go to EC2 dashboard
2. Click "Launch instances"
3. Configure the instance:
   - Name: `test-ec2-public`
   - AMI: Ubuntu
   - Instance type: t2.micro
   - Create new key pair: `nat-gateway-demo-key`
   - Network settings:
     - VPC: Select your test VPC
     - Subnet: Select public subnet
     - Auto-assign public IP: Enable
   - Security group: Allow SSH (port 22) and HTTP (port 80) from anywhere
4. Add user data script:
   ```bash
   #!/bin/bash
   apt-get update
   apt-get install -y apache2
   echo "$(hostname)" > /var/www/html/index.html
   systemctl restart apache2
   ```
5. Click "Launch instance"

#### Private EC2 Instance

1. Click "Launch instances"
2. Configure the instance:
   - Name: `test-ec2-private`
   - AMI: Ubuntu
   - Instance type: t2.micro
   - Key pair: Select the same key used for public instance
   - Network settings:
     - VPC: Select your test VPC
     - Subnet: Select private subnet
     - Auto-assign public IP: Disable
   - Security group: Allow SSH (port 22) and HTTP (port 80) from anywhere
3. Add the same user data script as the public instance
4. Click "Launch instance"

## Testing the Setup

### 1. Verify Public Instance Access

1. Go to EC2 dashboard
2. Select the public instance
3. Copy the public IP
4. Open in a web browser - you should see the hostname displayed

### 2. Access Private Instance via Public Instance

1. Change the permissions of your key file on your local machine:
   ```bash
   chmod 400 nat-gateway-demo-key.pem
   ```

2. SSH into the public instance:
   ```bash
   ssh -i nat-gateway-demo-key.pem ubuntu@<public-instance-ip>
   ```

3. Copy the private key to the public instance:
   - On your local machine, display the key:
     ```bash
     cat nat-gateway-demo-key.pem
     ```
   - Copy the content
   - On the public instance, create a file for the key:
     ```bash
     vi nat-gateway-demo-key.pem
     ```
   - Paste the content and save (`:wq`)
   - Change permissions:
     ```bash
     chmod 400 nat-gateway-demo-key.pem
     ```

4. From the public instance, SSH into the private instance:
   ```bash
   ssh -i nat-gateway-demo-key.pem ubuntu@<private-instance-private-ip>
   ```

### 3. Test Internet Access from Private Instance

1. While logged into the private instance, test internet connectivity:
   ```bash
   curl www.google.com
   ```
   
2. Try pinging Google:
   ```bash
   ping www.google.com
   ```

### 4. Verify NAT Gateway Functionality

To confirm the NAT Gateway is working correctly:

1. Go to VPC dashboard
2. Select "Route Tables"
3. Select the private route table
4. Click "Edit routes"
5. Remove the NAT Gateway route and save changes
6. Return to the private instance terminal
7. Try pinging Google again:
   ```bash
   ping www.google.com
   ```
   
The ping should now fail, confirming that the NAT Gateway was providing the internet access.

## Conclusion

This exercise demonstrates how to set up a NAT Gateway to allow EC2 instances in private subnets to access the internet for updates and package downloads while maintaining security by preventing direct inbound connections from the internet.

Key concepts learned:
- NAT Gateway provides one-way internet access for private resources
- NAT Gateway must be placed in a public subnet with an Internet Gateway
- Public and private route tables manage traffic flow differently
- NAT Gateway requires an Elastic IP address
- Proper security configuration allows for secure, controlled internet access
