# AWS Application Load Balancer (ALB) Setup

In this guide, I documented my process of setting up an Application Load Balancer in AWS, covering each step from creating a VPC to configuring EC2 instances and the load balancer.

## Understanding the Application Load Balancer

An Application Load Balancer operates at the application layer (Layer 7) of the OSI model, routing HTTP/HTTPS traffic to registered targets (e.g., EC2 instances) based on rules. I visualized the ALB as a traffic conductor, distributing incoming requests across multiple EC2 instances in different subnets to ensure high availability and scalability.

The architecture I aimed to build included:
- **VPC**: A virtual network to house all resources.
- **Internet Gateway**: To enable internet access.
- **Public Subnets**: Two subnets in different Availability Zones for EC2 instances.
- **Route Table**: To connect subnets to the Internet Gateway.
- **EC2 Instances**: Two instances running Apache servers, each in a different public subnet.
- **Target Group**: A logical group containing the EC2 instances.
- **Application Load Balancer**: To route traffic to the target group.

![image alt](https://github.com/FeEgyir/Understanding-AWS-Services/blob/d3ace5c5798233a5aa4b233d6fd5b1880c111b2a/All%20Images/Application%20%20Load%20Balancer.png)
```A diagram of the ALB architecture, showing a user accessing the ALB via the Internet Gateway, with traffic routed to two EC2 instances in separate public subnets within a VPC.```

## Step 1: Creating a VPC
I began by creating a Virtual Private Cloud (VPC) to serve as the network foundation.

1. Navigated to the VPC Dashboard in the AWS Console.
2. Clicked "Create VPC" → Selected "VPC only".
3. Configured:
  - Name: `test-VPC`
  - IPv4 CIDR block: `12.0.0.0/16` (65,536 IP addresses)
  - Tags: Name: `test-VPC`
4. Clicked Create VPC.

I confirmed the VPC’s creation in the VPC list, noting its CIDR block for subnet planning.

## Step 2: Attaching an Internet Gateway
To enable internet access for public subnets, I created and attached an Internet Gateway.

1. From the VPC Dashboard, selected Internet Gateways.
2. Clicked Create internet gateway.
  - Named it `test-IGW`.
  - Added tag Name: `test-IGW`.
3. Clicked Create.
4. Attached it to the VPC:
  - Selected `test-IGW`.
  - Clicked Actions > Attach to VPC.
  - Chose `test-VPC`.
5. Clicked Attach internet gateway.

I verified the attachment by checking the gateway’s state, ensuring it was “attached”.

## Step 3: Creating Public Subnets
I created two public subnets in different Availability Zones for high availability.

### Subnet 1 (eu-central-1a)

1. Navigated to Subnets in the VPC Dashboard.
2. Clicked Create subnet.
3. Selected test-VPC.
4. Configured:
  - Name: `test-public-subnet-1a`
  - Availability Zone: `eu-central-1a`
  - IPv4 CIDR block: `12.0.1.0/24` (256 IP addresses)

Clicked Add new subnet for the second subnet.

### Subnet 2 (eu-central-1b)

1. Configured:
  - Name: `test-public-subnet-1b`
  - Availability Zone: `eu-central-1b`
  - IPv4 CIDR block: `12.0.3.0/24`

Clicked Create subnet.

I ensured non-overlapping CIDR blocks and selected Availability Zones in the eu-central-1 region to support redundancy.

## Step 4: Configuring a Route Table
I set up a route table to connect the public subnets to the Internet Gateway.

1. Navigated to Route Tables in the VPC Dashboard.
2. Clicked Create route table.
3. Configured:
  - Name: `test-public-RT`
  - VPC: `test-VPC`
  - Tags: Name: `test-public-RT`
4. Clicked Create route table.
5. Associated it with both subnets:
  - Selected test-public-RT.
  - Went to Subnet Associations.
  - Clicked Edit subnet associations.
  - Selected test-public-subnet-1a and test-public-subnet-1b.
  - Clicked Save associations.
6. Added an internet route:
  - Went to Routes.
  - Clicked Edit routes.
  - Added:
    - Destination: `0.0.0.0/0`
    - Target: Internet Gateway > `test-IGW`
  - Clicked Save changes.

This allowed resources in the public subnets to access and be accessed from the internet.

![image alt](https://github.com/FeEgyir/Understanding-AWS-Services/blob/d3ace5c5798233a5aa4b233d6fd5b1880c111b2a/All%20Images/VPC.png)

## Step 5: Launching EC2 Instances
I launched two EC2 instances in separate subnets, each running an Apache server configured via a user data script.

### EC2 Instance 1 (Subnet 1a)

1. Navigated to the EC2 Dashboard.
2. Clicked Launch instance.
3. Configured:
  - Name: `test-ec2-instance-1`
  - AMI: Amazon Linux 2023 (x86_64, latest, free tier eligible)
  - Instance Type: t2.micro
  - Key Pair:
    - Created new: `test-ALB-demo-key-pair`
    - Downloaded .pem file securely
  - Network Settings:
  - VPC: `test-vpc`
  - Subnet: `test-public-subnet-1a`
  - Auto-assign Public IP: Enabled
  - Security Group: Created new; `test-ec2-SG`
  - Rules: This is to make them accessible from outside AWS environment
    - SSH (Port 22) from 0.0.0.0/0
    - HTTP (Port 80) from 0.0.0.0/0
  
  - Storage: Default (8 GiB gp2)
  - User Data:
```bash
#!/bin/bash
dnf update -y
dnf install httpd -y
systemctl start httpd
systemctl enable httpd
echo "<h1>Server: $(hostname)</h1>" > /var/www/html/index.html
```
4. Clicked Launch instance.

The user data script installs Apache, starts it, and creates an index.html file displaying the instance’s hostname.

![images alt](https://github.com/FeEgyir/Understanding-AWS-Services/blob/90a03af32b10840d0f0690a65144f13c33578407/All%20Images/1.png)

### EC2 Instance 2 (Subnet 1b)

1. Repeated for the second instance:
  - Name: `test-ec2-instance-2`
  - AMI: Ubuntu Server (x86_64, latest, free tier eligible)
  - Instance Type: `t2.micro`
  - Key Pair: `test-ALB-demo-key-pair`
  - Network Settings:
    - VPC: `test-VPC`
    - Subnet: `test-public-subnet-1b`
    - Auto-assign Public IP: Enabled
    - Security Group: Reused `test-ec2-SG`
  - Storage: Default
  - User Data:
```
#!/bin/bash
apt-get update -y
apt-get install apache2 -y
systemctl start apache2
systemctl enable apache2
echo "<h1>Server: $(hostname)</h1>" > /var/www/html/index.html
```
2. Clicked Launch instance.

![images alt](https://github.com/FeEgyir/Understanding-AWS-Services/blob/90a03af32b10840d0f0690a65144f13c33578407/All%20Images/2.png)

I waited for both instances to reach “running” status, then tested Apache by accessing each public IP in a browser, confirming hostnames and private IPs (e.g., `12.0.1.195`, `12.0.3.75`).

## Step 6: Creating a Target Group
I created a Target Group to group the EC2 instances for ALB routing.

1. From the EC2 Dashboard, went to Load Balancing > Target Groups.
2. Clicked Create target group.
3. Configured:
  - Target type: Instances
  - Name: `test-ec2-ALB-demo-TG`
  - Protocol: HTTP
  - Port: 80
  - VPC: `test-VPC`
  - Protocol version: HTTP1
  - Health check path: /
  - Advanced health check settings: Default
  - Tags: None
4. Clicked Next.
5. Selected both instances (`test-ec2-instance-1`, `test-ec2-instance-2`).
6. Set Port: 80.
7. Clicked Include as pending below.
8. Clicked Create target group.

The Target Group was created, initially showing “unused” health status until linked to the ALB.

## Step 7: Creating the Application Load Balancer
I configured the ALB to route traffic to the Target Group.

1. From the EC2 Dashboard, went to Load Balancing > Load Balancers.
2. Clicked Create load balancer.
3. Chose Application Load Balancer.
4. Configured:
  - Name: `test-LB-for-ec2-demo`
  - Scheme: Internet-facing
  - IP address type: IPv4
  - Network mapping:
    - VPC: `test-VPC`
    - Mappings: `test-public-subnet-1a`, `test-public-subnet-1b`
  - Security Groups:
    - Kept default security group
    - Created new:
      - Name: `security-group-for-ALB-demo`
      - Description: Allow access from internet
      - VPC: `test-VPC`
    - Rule: HTTP (Port 80) from `0.0.0.0/0`
  - Listeners and routing:
    - Protocol: HTTP
    - Port: 80
    - Default action: `test-ec2-ALB-demo-TG`
  - Tags: None
5. Verified the summary (VPC, subnets, security groups, Target Group).
6. Clicked Create load balancer.

![image alt](https://github.com/FeEgyir/Understanding-AWS-Services/blob/d3ace5c5798233a5aa4b233d6fd5b1880c111b2a/All%20Images/TG.png)

The ALB took a few minutes to transition from “provisioning” to “active”.

## Step 8: Testing the Application Load Balancer
I tested the ALB’s traffic distribution.

1. From the Load Balancers page, selected the ALB.
2. Copied its DNS name (e.g., `test-LB-for-ec2-demo-123456.elb.eu-central-1.amazonaws.com`).
3. Accessed the DNS name in a browser, refreshing multiple times.
4. Observed the page switching between the two EC2 instances’ homepages, showing their private IPs (`12.0.1.195`, `12.0.1.75`).

I confirmed the Target Group’s health status as “healthy” for both instances after ALB activation.

## Verifying the Setup
I validated:

- The ALB distributed traffic across both EC2 instances, alternating IPs in the browser.
- Each EC2 instance’s Apache server was accessible via its public IP.
- The Target Group’s health checks confirmed instance responsiveness.

[watch the video](https://github.com/user-attachments/assets/03f92851-389e-49dc-a84d-d43454a80ce2)


## Key Learnings
I gained insights into:

### ALB Role: 
- Distributes HTTP traffic for scalability and availability.

### Networking:
- Built a VPC with public subnets across Availability Zones.
- Configured Internet Gateway and route table for connectivity.

### EC2 Setup:
- Automated Apache installation on Ubuntu & Amazon Linux instances using OS-specific with user data.
- Secured instances with SSH and HTTP security group rules.

### Target Groups: 
- Enabled health checks and instance grouping.

### ALB Configuration:
- Set up an internet-facing ALB with proper listeners and security.
- Linked it to a Target Group for routing.

## Testing: Confirmed load balancing via DNS name access.
To enhance this setup, I could:

1. Add HTTPS with AWS Certificate Manager.
2. Implement Auto Scaling for dynamic instance management.
3. Monitor ALB and instances with CloudWatch.
4. Explore path-based routing for advanced traffic rules.
5. Test a Network Load Balancer for Layer 4 use cases.

## Troubleshooting Tips
If issues occur, I’d check:
1. Security Groups: Verify HTTP (Port 80) for ALB and EC2, SSH (Port 22) for EC2.
2. Route Table: Ensure 0.0.0.0/0 routes to the Internet Gateway and subnets are associated.
3. Target Group: Confirm “healthy” status (check Apache and health check path).
4. Subnets: Verify both are public and mapped to the ALB.
5. DNS: Ensure the ALB’s DNS name resolves correctly.

## Conclusion
Setting up an AWS Application Load Balancer was a valuable learning experience, reinforcing my understanding of load balancing and AWS networking. By creating a VPC, subnets, EC2 instances, a Target Group, and an ALB, I built a scalable web application infrastructure. This process equipped me with practical skills for implementing high-availability solutions in AWS.
