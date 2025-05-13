# AWS VPC Peering Implementation Guide

## Introduction

This document covers the implementation of AWS VPC peering between two VPCs. VPC peering allows network connectivity between two separate VPCs, enabling resources in each VPC to communicate with each other as if they were within the same network.

## Overview

In this implementation, I'll set up:
- Two separate VPCs (Test VPC 1 and Test VPC 2)
- A subnet in each VPC
- EC2 instances in each subnet
- VPC peering connection between the VPCs
- Appropriate route table configurations

![VPC Peering Architecture Diagram](https://placeholder-image.com/vpc-peering-architecture.png)

## Prerequisites

- An AWS account
- Access to VPC and EC2 services
- Basic knowledge of networking concepts

## Step 1: Create the VPCs

First, I need to create two separate VPCs:

1. Navigate to VPC in the AWS Console
2. Click "Create VPC"
3. For the first VPC:
   - Select "VPC only"
   - Name: Test VPC 1
   - CIDR block: 12.0.0.0/16 (provides approximately 65,000 IP addresses)
   - Click "Create VPC"
4. For the second VPC:
   - Select "VPC only"
   - Name: Test VPC 2
   - CIDR block: 13.0.0.0/16
   - Click "Create VPC"

## Step 2: Create Route Tables

Next, I'll create route tables for each VPC:

1. In the VPC dashboard, select "Route Tables" from the left menu
2. Click "Create route table"
3. For the first route table:
   - Name: Test RT for VPC 1
   - VPC: Select Test VPC 1
   - Click "Create route table"
4. For the second route table:
   - Name: Test RT for VPC 2
   - VPC: Select Test VPC 2
   - Click "Create route table"

## Step 3: Create Subnets

Now I'll create subnets within each VPC:

1. In the VPC dashboard, select "Subnets" from the left menu
2. Click "Create subnet"
3. For the first subnet:
   - Select VPC: Test VPC 1
   - Subnet name: Test subnet VPC 1
   - Availability Zone: Select an AZ (e.g., eu-central-1a)
   - IPv4 CIDR block: 12.0.1.0/24 (provides 256 IP addresses)
   - Click "Create subnet"
4. For the second subnet:
   - Select VPC: Test VPC 2
   - Subnet name: Test subnet VPC 2
   - Availability Zone: Select the same AZ as the first subnet
   - IPv4 CIDR block: 13.0.1.0/24
   - Click "Create subnet"

## Step 4: Associate Route Tables with Subnets

Now I'll associate the route tables with their respective subnets:

1. Go back to "Route Tables"
2. Select the route table for VPC 1
3. Go to "Subnet Associations" tab
4. Click "Edit subnet associations"
5. Select the subnet for VPC 1
6. Click "Save associations"
7. Repeat for the route table for VPC 2, associating it with the subnet for VPC 2

## Step 5: Create Internet Gateways

To allow internet access, I need to create Internet Gateways:

1. In the VPC dashboard, select "Internet Gateways"
2. Click "Create internet gateway"
3. For the first gateway:
   - Name: Test Internet Gateway for VPC 1
   - Click "Create internet gateway"
   - Click "Actions" > "Attach to VPC"
   - Select Test VPC 1
   - Click "Attach internet gateway"
4. For the second gateway:
   - Name: Test Internet Gateway for VPC 2
   - Click "Create internet gateway"
   - Click "Actions" > "Attach to VPC"
   - Select Test VPC 2
   - Click "Attach internet gateway"

## Step 6: Update Route Tables for Internet Access

Now I'll update the route tables to allow internet access:

1. Go back to "Route Tables"
2. Select the route table for VPC 1
3. Go to the "Routes" tab
4. Click "Edit routes"
5. Click "Add route"
6. For Destination, enter: 0.0.0.0/0
7. For Target, select "Internet Gateway" and choose the Test Internet Gateway for VPC 1
8. Click "Save changes"
9. Repeat for the route table for VPC 2, selecting the Internet Gateway for VPC 2

## Step 7: Launch EC2 Instances

Now I'll create EC2 instances in each VPC:

### For the First EC2 Instance (in VPC 1):

1. Navigate to EC2 in the AWS Console
2. Click "Launch instances"
3. Enter a name: VPC1 EC2 Instance
4. Select an Amazon Machine Image (AMI): Ubuntu
5. Choose instance type: t2.micro
6. Create a new key pair: VPC1 EC2 Key
7. Network settings:
   - VPC: Select Test VPC 1
   - Subnet: Select Test subnet VPC 1
   - Auto-assign public IP: Enable
8. Security group rules:
   - Add rule for SSH (port 22) from anywhere
   - Add rule for HTTP (port 80) from anywhere
9. Configure storage: 8 GB
10. Advanced details - User data:
    ```bash
    #!/bin/bash
    apt-get update
    apt-get install -y apache2
    echo "Server Details: $(hostname)" > /var/www/html/index.html
    systemctl restart apache2
    ```
11. Click "Launch instance"

### For the Second EC2 Instance (in VPC 2):

1. Click "Launch instances"
2. Enter a name: VPC2 EC2 Instance
3. Select an Amazon Machine Image (AMI): Ubuntu
4. Choose instance type: t2.micro
5. Create a new key pair: VPC2 EC2 Key
6. Network settings:
   - VPC: Select Test VPC 2
   - Subnet: Select Test subnet VPC 2
   - Auto-assign public IP: Enable
7. Security group rules:
   - Add rule for SSH (port 22) from anywhere
   - Add rule for HTTP (port 80) from anywhere
8. Configure storage: 8 GB
9. Advanced details - User data: Same script as the first instance
10. Click "Launch instance"

## Step 8: Verify Apache Installation

To confirm the Apache installation is working:

1. Go to the EC2 dashboard
2. Select the first instance and copy its public IP address
3. Paste the IP address into a browser
4. Verify that the Apache page displays with the server details
5. Repeat for the second instance

## Step 9: Create VPC Peering Connection

Now I'll establish the VPC peering connection:

1. Return to the VPC dashboard
2. Select "Peering Connections" from the left menu
3. Click "Create peering connection"
4. Configure peering connection:
   - Name: Peering-Connection-VPC1-VPC2
   - Requester VPC: Select Test VPC 1
   - Accepter VPC: Select Test VPC 2
5. Click "Create peering connection"
6. Select the pending peering connection
7. From the "Actions" menu, select "Accept request"
8. Confirm the details and click "Accept request"

## Step 10: Update Route Tables for VPC Peering

To enable communication between the VPCs, I need to update the route tables:

1. Go to "Route Tables"
2. Select the route table for VPC 1
3. Go to the "Routes" tab
4. Click "Edit routes"
5. Click "Add route"
6. For Destination, enter the CIDR block of VPC 2 (13.0.0.0/16)
7. For Target, select "Peering Connection" and choose the peering connection created
8. Click "Save changes"
9. Select the route table for VPC 2
10. Go to the "Routes" tab
11. Click "Edit routes"
12. Click "Add route"
13. For Destination, enter the CIDR block of VPC 1 (12.0.0.0/16)
14. For Target, select "Peering Connection" and choose the peering connection created
15. Click "Save changes"

## Step 11: Test the VPC Peering Connection

To test the connection between the VPCs:

1. SSH into the EC2 instance in VPC 1:
   ```bash
   chmod 400 VPC1-EC2-Key.pem
   ssh -i "VPC1-EC2-Key.pem" ubuntu@<public-ip>
   ```

2. From the VPC 1 instance, use curl to access the Apache page on the VPC 2 instance:
   ```bash
   curl <private-ip-of-vpc2-instance>
   ```

3. SSH into the EC2 instance in VPC 2:
   ```bash
   chmod 400 VPC2-EC2-Key.pem
   ssh -i "VPC2-EC2-Key.pem" ubuntu@<public-ip>
   ```

4. From the VPC 2 instance, use curl to access the Apache page on the VPC 1 instance:
   ```bash
   curl <private-ip-of-vpc1-instance>
   ```

If both curl commands display the Apache page, the VPC peering connection is working correctly.

## Verification Test: Remove Route Table Entries

To verify that the VPC peering is indeed enabling the communication:

1. Go to "Route Tables"
2. Select the route table for VPC 1
3. Go to the "Routes" tab
4. Click "Edit routes"
5. Remove the route entry for the VPC 2 CIDR block
6. Click "Save changes"
7. Repeat for the route table for VPC 2, removing the route entry for the VPC 1 CIDR block

Now testing the curl commands again should fail, confirming that the VPC peering connection was indeed responsible for enabling the communication.

## Benefits of VPC Peering

- Allows resources in different VPCs to communicate as if they were in the same network
- Maintains network isolation between VPCs except for the specifically allowed traffic
- Provides higher bandwidth and lower latency than routing through the internet
- No single point of failure or bandwidth bottleneck
- Traffic remains within the AWS network and doesn't traverse the internet

## Limitations and Considerations

- VPC peering does not support transitive peering relationships
- CIDR blocks cannot overlap between peered VPCs
- Security groups from one VPC cannot be referenced in the other VPC
- VPC peering connections can be established between VPCs in different regions (inter-region VPC peering)
