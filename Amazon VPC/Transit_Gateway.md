# AWS Transit Gateway Setup

## Overview
In this guide, I'm documenting how to set up an AWS Transit Gateway to enable communication between multiple VPCs within the same AWS account. Transit Gateway acts as a network transit hub that allows multiple VPCs to communicate with each other without the complexity of managing numerous VPC peering connections.

## Architecture Overview
The setup consists of:
- 3 VPCs (test-vpc-1, test-vpc-2, test-vpc-3)
- Each VPC will have a public subnet
- Each subnet will contain an EC2 instance running Apache
- 1 Transit Gateway that connects all 3 VPCs

## IP Ranges Used
- VPC 1: 12.0.0.0/16 (Subnet: 12.0.1.0/24)
- VPC 2: 13.0.0.0/16 (Subnet: 13.0.1.0/24)
- VPC 3: 14.0.0.0/16 (Subnet: 14.0.1.0/24)

## Why Transit Gateway vs VPC Peering?
A key concept to understand is why Transit Gateway is more efficient than VPC peering when connecting multiple VPCs:

- **VPC Peering**: Requires a separate connection between each pair of VPCs. For N VPCs, you need N(N-1)/2 peering connections. This becomes unwieldy with more than a few VPCs.
- **Transit Gateway**: Requires just one connection from each VPC to the Transit Gateway. For N VPCs, you need only N connections.

## Step 1: Creating VPC 1

1. Navigate to the AWS console and search for VPC
2. Click on "Create VPC"
3. Enter the following details:
   - Name: test-vpc-1
   - IPv4 CIDR block: 12.0.0.0/16
   - Keep other settings as default
4. Click "Create VPC"

## Step 2: Create Internet Gateway for VPC 1

1. In the VPC dashboard, go to "Internet Gateways" in the left navigation menu
2. Click "Create internet gateway"
3. Enter "igw-test-vpc-1" (naming with vpc suffix for identification)
4. Click "Create internet gateway"
5. Once created, attach it to VPC 1:
   - Select the internet gateway
   - Go to "Actions" > "Attach to VPC"
   - Select "test-vpc-1"
   - Click "Attach internet gateway"

## Step 3: Create Subnet for VPC 1

1. In the VPC dashboard, go to "Subnets" in the left navigation menu
2. Click "Create subnet"
3. Select "test-vpc-1" for the VPC
4. Enter "test-subnet-vpc-1-1a" as the name
5. Select an Availability Zone (e.g., eu-central-1a)
6. Enter IPv4 CIDR block: 12.0.1.0/24
7. Click "Create subnet"

## Step 4: Create Route Table for VPC 1

1. In the VPC dashboard, go to "Route Tables" in the left navigation menu
2. Click "Create route table"
3. Enter "rt-test-vpc-1" as the name
4. Select "test-vpc-1" for the VPC
5. Click "Create route table"
6. Associate the subnet with the route table:
   - Select the created route table
   - Go to "Subnet Associations" tab
   - Click "Edit subnet associations"
   - Select "test-subnet-vpc-1-1a"
   - Click "Save associations"
7. Add an internet route:
   - Go to "Routes" tab
   - Click "Edit routes"
   - Click "Add route"
   - Enter "0.0.0.0/0" as the destination
   - Select "Internet Gateway" and choose "igw-test-vpc-1"
   - Click "Save changes"

## Step 5: Create EC2 Instance in VPC 1

1. Navigate to EC2 in the AWS console
2. Click "Launch instance"
3. Enter "ec2-test-vpc-1" as the name
4. Select Ubuntu as the AMI
5. Choose t2.micro as the instance type
6. Create a new key pair "ec2-test-vpc-1-key"
7. Configure network settings:
   - Select "test-vpc-1" as the VPC
   - Select "test-subnet-vpc-1-1a" as the subnet
   - Enable "Auto-assign public IP"
   - Create a security group allowing SSH (port 22) and HTTP (port 80) from anywhere
8. Add user data to install Apache:
```bash
#!/bin/bash
apt-get update -y
apt-get install -y apache2
echo "<h1>Host Name: $(hostname -f)</h1><h3>Private IP: $(hostname -I | awk '{print $1}')</h3>" > /var/www/html/index.html
systemctl restart apache2
```
9. Click "Launch instance"

## Step 6: Repeat for VPC 2 and VPC 3

Follow the same steps as above for VPC 2 and VPC 3, changing the names and IP ranges:

For VPC 2:
- VPC CIDR: 13.0.0.0/16
- Subnet CIDR: 13.0.1.0/24
- Names: append "-vpc-2" instead of "-vpc-1"

For VPC 3:
- VPC CIDR: 14.0.0.0/16
- Subnet CIDR: 14.0.1.0/24
- Names: append "-vpc-3" instead of "-vpc-1"

## Step 7: Create Transit Gateway

1. Navigate to VPC in the AWS console
2. Scroll down to "Transit Gateways" in the left navigation menu
3. Click "Create Transit Gateway"
4. Enter "tgw-vpc1-vpc2-vpc3" as the name
5. Add description: "Transit Gateway for VPC 1, VPC 2, and VPC 3"
6. Leave ASN as default (AWS will assign one)
7. Leave cross-account sharing options and CIDR blocks as default
8. Click "Create Transit Gateway"
9. Wait for the Transit Gateway to become available (this may take a few minutes)

## Step 8: Create Transit Gateway Attachments

### For VPC 1:
1. Go to "Transit Gateway Attachments" in the left navigation menu
2. Click "Create Transit Gateway Attachment"
3. Enter "transit-gateway-attachment-vpc-1" as the name
4. Select the transit gateway created earlier
5. Select "VPC" as the attachment type
6. Select "test-vpc-1" as the VPC
7. Select the subnet "test-subnet-vpc-1-1a"
8. Click "Create Transit Gateway Attachment"

### For VPC 2:
1. Click "Create Transit Gateway Attachment"
2. Enter "transit-gateway-attachment-vpc-2" as the name
3. Select the same transit gateway
4. Select "VPC" as the attachment type
5. Select "test-vpc-2" as the VPC
6. Select the subnet "test-subnet-vpc-2-1a"
7. Click "Create Transit Gateway Attachment"

### For VPC 3:
1. Click "Create Transit Gateway Attachment"
2. Enter "transit-gateway-attachment-vpc-3" as the name
3. Select the same transit gateway
4. Select "VPC" as the attachment type
5. Select "test-vpc-3" as the VPC
6. Select the subnet "test-subnet-vpc-3-1a"
7. Click "Create Transit Gateway Attachment"

## Step 9: Update Route Tables

### For VPC 1 Route Table:
1. Go to "Route Tables" in the VPC dashboard
2. Select the route table for VPC 1
3. Go to "Routes" tab and click "Edit routes"
4. Add a route:
   - Destination: 13.0.0.0/16 (VPC 2 CIDR)
   - Target: Select Transit Gateway
   - Click "Save changes"
5. Add another route:
   - Destination: 14.0.0.0/16 (VPC 3 CIDR)
   - Target: Select Transit Gateway
   - Click "Save changes"

### For VPC 2 Route Table:
1. Select the route table for VPC 2
2. Go to "Routes" tab and click "Edit routes"
3. Add a route:
   - Destination: 12.0.0.0/16 (VPC 1 CIDR)
   - Target: Select Transit Gateway
   - Click "Save changes"
4. Add another route:
   - Destination: 14.0.0.0/16 (VPC 3 CIDR)
   - Target: Select Transit Gateway
   - Click "Save changes"

### For VPC 3 Route Table:
1. Select the route table for VPC 3
2. Go to "Routes" tab and click "Edit routes"
3. Add a route:
   - Destination: 12.0.0.0/16 (VPC 1 CIDR)
   - Target: Select Transit Gateway
   - Click "Save changes"
4. Add another route:
   - Destination: 13.0.0.0/16 (VPC 2 CIDR)
   - Target: Select Transit Gateway
   - Click "Save changes"

## Step 10: Testing Connectivity

1. Connect to the EC2 instance in VPC 1:
   ```bash
   chmod 400 ec2-test-vpc-1-key.pem
   ssh -i "ec2-test-vpc-1-key.pem" ubuntu@[public-ip-vpc-1-ec2]
   ```

2. Connect to the EC2 instance in VPC 2:
   ```bash
   chmod 400 ec2-test-vpc-2-key.pem
   ssh -i "ec2-test-vpc-2-key.pem" ubuntu@[public-ip-vpc-2-ec2]
   ```

3. Connect to the EC2 instance in VPC 3:
   ```bash
   chmod 400 ec2-test-vpc-3-key.pem
   ssh -i "ec2-test-vpc-3-key.pem" ubuntu@[public-ip-vpc-3-ec2]
   ```

4. Test connectivity between instances:
   - From VPC 1 EC2, curl the Apache page of VPC 2 EC2:
     ```bash
     curl [private-ip-vpc-2-ec2]
     ```
   - From VPC 1 EC2, curl the Apache page of VPC 3 EC2:
     ```bash
     curl [private-ip-vpc-3-ec2]
     ```
   - From VPC 2 EC2, curl VPC 1 and VPC 3 EC2
   - From VPC 3 EC2, curl VPC 1 and VPC 2 EC2

If all curl commands return the Apache homepage with the hostname and private IP information, then the Transit Gateway is successfully connecting all VPCs.

## Conclusion

I've successfully set up a Transit Gateway that connects three VPCs, allowing EC2 instances in each VPC to communicate with instances in the other VPCs. This approach is much more scalable than using VPC peering, especially when dealing with more than a few VPCs.

Transit Gateway serves as a hub that simplifies network architecture and reduces the number of connections required for inter-VPC communication. Instead of managing multiple peering connections (which would grow exponentially with the number of VPCs), we only need to manage one connection per VPC to the Transit Gateway.
