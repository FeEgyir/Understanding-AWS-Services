# AWS EC2 Auto Scaling Setup

This guide documents my process of learning and implementing an EC2 Auto Scaling group in AWS, including a VPC, subnets, internet gateway, route tables, application load balancer, target group, launch template, and the Auto Scaling group itself. The goal is to understand how Auto Scaling optimizes resource usage based on demand while maintaining high availability.

## Understanding EC2 Auto Scaling

Auto Scaling automatically adjusts the number of EC2 instances based on traffic, CPU, or memory usage. Key parameters include:

- **Minimum**: The least number of EC2 instances always running.
- **Maximum**: The highest number of instances allowed.
- **Desired**: The preferred number of instances under normal conditions.

Auto Scaling reduces costs compared to a traditional setup with a fixed number of instances. For example, in a traditional setup with three EC2 instances behind a load balancer, all three run regardless of demand, incurring unnecessary costs during low traffic (e.g., nighttime). Auto Scaling dynamically provisions or terminates instances based on load, ensuring efficiency.

## Target Architecture

The architecture includes:

- A VPC with two public subnets in different availability zones.
- An internet gateway attached to the VPC.
- A route table for public subnets with a route to the internet gateway.
- An application load balancer (ALB) routing traffic to a target group.
- A target group pointing to EC2 instances.
- An Auto Scaling group using a launch template to provision EC2 instances running Apache.
- Security groups for the ALB and EC2 instances to allow HTTP and SSH traffic.

## Step-by-Step Implementation

### 1. Create a VPC

To isolate resources, I created a VPC named test-vpc with an IPv4 CIDR block of 10.0.0.0/16.

1. Navigated to the AWS Console > VPC > Create VPC.
2. Selected "VPC only".
3. Set Name: `test-vpc`.
4. Set IPv4 CIDR: `10.0.0.0/16`.
5. Kept other settings as default and clicked "Create VPC".

The VPC appeared in the VPC dashboard after creation.

### 2. Create and Attach an Internet Gateway

An internet gateway enables internet access for the VPC.

1. Went to VPC > Internet Gateways > Create Internet Gateway.
2. Named it `internet-gateway-test`.
3. Clicked "Create Internet Gateway".
4. Selected the gateway, chose "Attach to VPC" from the Actions menu, and attached it to test-vpc.

### 3. Create Public Subnets

I created two public subnets in different availability zones for high availability.

1. Navigated to VPC > Subnets > Create Subnet.
2. Selected `test-vpc` from the dropdown.
3. Created the first subnet:
   - Name: `test-public-subnet-1a`.
   - Availability Zone: `eu-central-1a`.
   - IPv4 CIDR: `10.0.1.0/24` (provides 256 IP addresses).

4. Added a second subnet:
   - Name: `test-public-subnet-1b`.
   - Availability Zone: `eu-central-1b`.
   - IPv4 CIDR: `10.0.3.0/24`.

5. Clicked "Create Subnet".

### 4. Create and Configure a Route Table

A route table directs traffic from subnets to the internet gateway.

1. Went to VPC > Route Tables > Create Route Table.
2. Named it `route-table-test-public`.
3. Selected test-vpc and clicked "Create Route Table".
4. Associated the route table with subnets:
   - Selected the route table, went to "Subnet Associations" > "Edit Subnet Associations".
   - Checked both test-public-subnet-1a and test-public-subnet-1b, then saved.

5. Added a route for internet access:
   - Went to "Routes" > "Edit Routes" > "Add Route".
   - Set Destination: 0.0.0.0/0 (all traffic).
   - Set Target: internet-gateway-test.
   - Saved changes.

### 5. Create a Target Group

The target group will later point to EC2 instances created by the Auto Scaling group.

1. Navigated to EC2 > Target Groups > Create Target Group.
2. Chose "Instances" as the target type.
3. Named it `target-ec2-apache`.
4. Set Protocol: HTTP, Port: 80.
5. Selected `test-vpc` as the VPC.
6. Kept health check settings as default and clicked "Next".
7. Skipped registering targets (instances will be added by Auto Scaling) and clicked "Create Target Group".

### 6. Create an Application Load Balancer

The ALB routes traffic to the target group.

1. Went to EC2 > Load Balancers > Create Load Balancer.
2. Chose "Application Load Balancer".
3. Configured:
   - Name: `alb-ec2-instances-with-autoscaling-group`.
   - Scheme: `Internet-facing`.
   - IP address type: `IPv4`.
   - VPC: `test-vpc`.
   - Mappings: Selected both test-public-subnet-1a and test-public-subnet-1b.

4. Created a security group for the ALB:
   - Clicked "Create new security group".
   - Named it `alb-security-group-for-http`.
   - Description: Allow HTTP requests.
   - VPC: `test-vpc`.
   - Added rule: HTTP, Port 80, Source: `0.0.0.0/0`.
   - Created the security group.

5. Selected the new security group and the default security group for the ALB.
6. Configured listener:
   - Protocol: HTTP, Port: 80.
   - Forward to: `target-ec2-apache`.

7. Reviewed the summary and clicked "Create Load Balancer".
8. Waited a few minutes for the load balancer to become active.

### 7. Create a Launch Template

The launch template defines the EC2 instance configuration for the Auto Scaling group.

1. Navigated to EC2 > Auto Scaling Groups > Create Auto Scaling Group.
2. Clicked the link to create a new launch template.
3. Configured:
   - Name: `lt-ec2-instances-apache2`.
   - AMI: Selected Ubuntu (e.g., Ubuntu Server 20.04 LTS).
   - Instance type: `t2.micro`.
   - Key pair: Selected an existing key pair or created a new one.
   - Network settings:
     - Did not select subnets (subnets are specified in the Auto Scaling group).
     - Auto-assign public IP: Enabled.

4. Created a security group:
   - Navigated to VPC > Security Groups > Create Security Group.
   - Named it `launch-template-security-group-ec2-instances-apache2`.
   - Description: Allow SSH and HTTP requests.
   - VPC: `test-vpc`.
   - Rules:
     - HTTP, Port 80, Source: 0.0.0.0/0.
     - SSH, Port 22, Source: 0.0.0.0/0.

5. Created the security group.
6. Returned to the launch template and selected the new security group.
7. Storage: Kept default (8 GiB gp2 volume).
8. Advanced Details > User Data: Added a script to install Apache and display the instance's hostname:

```bash
#!/bin/bash
apt-get update
apt-get install -y apache2
echo "<h1>Server Details: $(hostname -f)</h1>" > /var/www/html/index.html
systemctl enable apache2
systemctl start apache2
```

9. Clicked "Create Launch Template".

### 8. Create the Auto Scaling Group

The Auto Scaling group uses the launch template to provision instances.

1. Returned to EC2 > Auto Scaling Groups > Create Auto Scaling Group.
2. Named it `autoscaling-group-ec2-instances-test-demo`.
3. Selected the launch template lt-ec2-instances-apache2 (version 1).
4. Configured:
   - VPC: `test-vpc`.
   - Availability Zones: Selected `eu-central-1a` and `eu-central-1b`.

5. Load balancing:
   - Chose "Attach to an existing load balancer".
   - Selected target-ec2-apache as the target group.

6. Enabled "Turn on Elastic Load Balancing health checks".
7. Set Health check grace period: 20 seconds (for demo; typically 300 seconds).
8. Set capacities:
   - Desired: 2 instances.
   - Minimum: 1 instance.
   - Maximum: 3 instances.

9. Scaling policies: Kept as "None" for simplicity (could add CPU-based scaling later).
10. Skipped notifications and tags.
11. Clicked "Create Auto Scaling Group".

### 9. Verify the Setup

After creating the Auto Scaling group, it provisioned two EC2 instances (based on the desired capacity).

1. Navigated to EC2 > Instances.
2. Confirmed two instances were running with status checks passed.
3. Went to EC2 > Load Balancers, copied the ALB's DNS name, and opened it in a browser.
4. Observed the Apache page displaying the hostname, which changed on refresh, indicating load balancing between instances.
5. Verified private IP addresses in EC2 > Instances matched the hostnames displayed.

### 10. Test Auto Scaling Behavior

To understand Auto Scaling's responsiveness:

1. Terminated one EC2 instance:
   - Went to EC2 > Instances, selected one instance, and chose "Terminate".

2. Checked the Auto Scaling group:
   - Navigated to EC2 > Auto Scaling Groups > selected the group > Instance Management.
   - After a minute, observed a new instance in the "Pending" state, as the group maintained the desired capacity of two.

3. Verified the target group:
   - Went to EC2 > Target Groups > target-ec2-apache.
   - Noted one instance was temporarily unhealthy, but the new instance became healthy after initialization.

## Key Learnings

- **Auto Scaling Efficiency**: Auto Scaling optimizes costs by adjusting instance counts based on demand, unlike a static setup.
- **High Availability**: Using multiple subnets in different availability zones ensures resilience.
- **Load Balancing**: The ALB distributes traffic evenly, and health checks ensure only healthy instances receive requests.
- **Launch Template**: Centralizes instance configuration, making it reusable for Auto Scaling.
- **Security Groups**: Properly configuring HTTP and SSH access is critical for functionality and security.
- **User Data**: Bootstrapping scripts automate setup tasks like installing Apache.

This setup provides a scalable, cost-effective web server environment. Next steps could include adding scaling policies based on CPU usage or exploring advanced health check configurations.
