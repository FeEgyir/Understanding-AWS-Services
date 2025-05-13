# Setting Up AWS WAF with Application Load Balancer

This document covers my process of setting up AWS Web Application Firewall (WAF) along with an Application Load Balancer to control traffic to EC2 instances. I'll explore how to allow and block traffic using WAF rules.

## Architecture Overview

The architecture consists of:
- VPC
- Internet Gateway
- Public Subnets in different availability zones
- Route Table with internet access
- EC2 instance running Apache
- Application Load Balancer
- AWS WAF for traffic filtering

## Step 1: Create a VPC

First, I needed to create a VPC to isolate my resources:

1. Navigated to AWS Console > VPC > Create VPC
2. Selected "VPC only" option
3. Set the name to "test-vpc"
4. Set IPv4 CIDR block to 12.0.0.0/16 (providing approximately 65,000 IP addresses)
5. Kept tenancy and tags as default
6. Clicked "Create VPC"

After creation, I verified the VPC appeared in the VPC dashboard.

## Step 2: Create and Attach an Internet Gateway

To provide internet access to the VPC:

1. In the VPC dashboard, selected Internet Gateways from the left navigation menu
2. Clicked "Create Internet Gateway"
3. Named it "internet-gway-igw-test"
4. Clicked "Create Internet Gateway"
5. Selected the newly created gateway
6. Clicked Actions > "Attach to VPC"
7. Selected the test-vpc
8. Clicked "Attach Internet Gateway"

## Step 3: Create Public Subnets

I needed to create two public subnets in different availability zones for high availability:

1. From the VPC dashboard, selected Subnets from the left navigation menu
2. Clicked "Create Subnet"
3. Selected test-vpc
4. For the first subnet:
   - Named it "test-public-subnet-1a" (1a suffix identifies the availability zone)
   - Selected eu-central-1a as the Availability Zone
   - Set IPv4 CIDR block to 12.0.1.0/24 (providing 256 IP addresses)
5. Clicked "Add new subnet" for the second subnet:
   - Named it "test-public-subnet-1b" 
   - Selected eu-central-1b as the Availability Zone
   - Set IPv4 CIDR block to 12.0.2.0/24
6. Clicked "Create Subnet"

## Step 4: Create and Configure a Route Table

To connect the internet gateway to the subnets:

1. From the VPC dashboard, selected Route Tables
2. Clicked "Create Route Table"
3. Named it "test-public-RT" (RT suffix indicates it's a route table)
4. Selected test-vpc
5. Clicked "Create Route Table"
6. After creation, associated subnets to the route table:
   - Selected the route table
   - Clicked "Subnet Associations" > "Edit Subnet Associations"
   - Selected both test-public-subnet-1a and test-public-subnet-1b
   - Clicked "Save Associations"
7. Added a route for internet access:
   - Selected the route table
   - Clicked "Routes" > "Edit Routes"
   - Clicked "Add Route"
   - Set Destination to 0.0.0.0/0 (all traffic)
   - Set Target to the internet gateway (internet-gway-igw-test)
   - Clicked "Save Changes"

## Step 5: Create an EC2 Instance

Now I needed to create an EC2 instance to be protected by the WAF:

1. Navigated to EC2 dashboard
2. Clicked "Launch Instance"
3. Set name to "test-ec2-AWS-WAF-demo"
4. Selected Ubuntu as the AMI
5. Chose t2.micro as the instance type
6. Created a new key pair named "AWS-WAF-ec2-demo"
7. Modified network settings:
   - Selected test-vpc as the VPC
   - Selected test-public-subnet-1a as the subnet
   - Enabled auto-assign public IP
   - Created security groups for:
     - SSH (port 22) from anywhere
     - HTTP (port 80) from anywhere
8. Kept default storage (8 GB)
9. Added user data script to install and configure Apache:

```bash
#!/bin/bash
apt-get update
apt-get install -y apache2
echo "<h1>Server Details: $(hostname) $(hostname -I)</h1>" > /var/www/html/index.html
systemctl restart apache2
```

10. Clicked "Launch Instance"
11. Waited for the instance to enter the "running" state
12. Verified access by navigating to the EC2 instance's public IP address in a browser

## Step 6: Create a Target Group

Before creating the load balancer, I needed to create a target group:

1. From the EC2 dashboard, selected Target Groups from the left navigation menu
2. Clicked "Create Target Group"
3. Selected "Instances" as the target type
4. Named the target group "Target-group-ec2-instances"
5. Kept Protocol as HTTP and Port as 80
6. Selected test-vpc as the VPC
7. Kept default health check settings
8. Clicked "Next"
9. Selected my EC2 instance to include in the target group
10. Clicked "Include as pending below"
11. Clicked "Create Target Group"

## Step 7: Create an Application Load Balancer

Next, I created an Application Load Balancer to distribute traffic:

1. From the EC2 dashboard, selected Load Balancers from left navigation menu
2. Clicked "Create Load Balancer"
3. Selected "Application Load Balancer"
4. Configured basic settings:
   - Named it "lb-WAF-demo"
   - Selected "Internet-facing" scheme
   - Selected IPv4 address type
   - Selected test-vpc
   - Selected both test-public-subnet-1a and test-public-subnet-1b for mappings
5. Created a new security group:
   - Named it "AWS-WAF-demo-Security-Group"
   - Description: "HTTP requests"
   - Selected test-vpc
   - Added rules:
     - SSH (port 22) from anywhere
     - HTTP (port 80) from anywhere
   - Created the security group
6. Selected the newly created security group for the load balancer
7. Configured listener and routing:
   - Protocol: HTTP, Port: 80
   - Forward to: Target-group-ec2-instances
8. Kept default settings for remaining options
9. Clicked "Create Load Balancer"
10. Waited for the load balancer to become active
11. Verified by navigating to the load balancer's DNS name in a browser

## Step 8: Create IP Set for WAF Rules

Before creating the WAF, I needed to set up an IP set to define which IPs to block:

1. Navigated to AWS WAF & Shield from AWS homepage
2. Selected the appropriate region (Europe Frankfurt)
3. From the left navigation menu, selected "IP sets"
4. Clicked "Create IP set"
5. Named it "my-laptop-IP"
6. Selected the region (Europe Frankfurt)
7. Found my IP address (by searching "what is my IP" on Google)
8. Entered my IP address with /32 suffix (for exact match)
9. Clicked "Create IP set"

## Step 9: Create AWS WAF

Now I set up the Web Application Firewall:

1. In the WAF & Shield dashboard, selected "Web ACLs" from left navigation menu
2. Clicked "Create web ACL"
3. Configured basic settings:
   - Named it "AWS-WAF-demo-ec2"
   - Added description
   - Selected "Regional resources"
   - Confirmed region as Europe Frankfurt
4. Added resources to protect:
   - Selected "Application Load Balancer"
   - Selected my load balancer (lb-WAF-demo)
   - Clicked "Add"
5. Added rules:
   - Selected "Add my own rules and rule groups"
   - Selected "IP set" rule type
   - Named the rule "block-my-laptop-access"
   - Selected "my-laptop-IP" IP set
   - Selected "Source IP address" option
   - Set action to "Block"
   - Clicked "Add rule"
6. Set default action to "Allow" for remaining requests
7. Clicked "Next" through remaining screens, keeping default settings
8. Clicked "Create web ACL"
9. Waited for the WAF to be created
10. Verified the WAF was properly configured:
    - Checked that the load balancer was associated
    - Confirmed the rules were in place

## Step 10: Test the WAF Rules

After setting up, I tested the WAF functionality:

1. Tested blocking:
   - Navigated to the load balancer's DNS name in my browser
   - Observed a 403 Forbidden error, confirming my IP was blocked
2. Changed rule to allow traffic:
   - Returned to the WAF console
   - Selected my rule
   - Clicked "Edit rule"
   - Changed action from "Block" to "Allow"
   - Saved the rule
   - Refreshed the browser page and confirmed I could access the EC2 instance
3. Tested CAPTCHA challenge:
   - Changed the rule action to "CAPTCHA"
   - Refreshed the browser page
   - Observed a CAPTCHA challenge appeared before access was granted

## Conclusion

I've successfully set up an AWS Web Application Firewall with an Application Load Balancer to control access to EC2 instances. The WAF allows me to define rules based on IP addresses and implement different actions (block, allow, CAPTCHA) for incoming traffic. This provides an additional security layer beyond traditional security groups and enables more sophisticated traffic filtering.
