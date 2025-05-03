# Understanding and Implementing Bastion Hosts in AWS

## Introduction to Bastion Hosts

I wanted to understand the concept of a Bastion host in AWS, so I started by thinking about it through a simple analogy before diving into the technical implementation.

Imagine a castle that can only be accessed through a heavily guarded tunnel. Anyone wanting to enter must pass through this fortified tunnel first. People with valid access can pass through without problems, but intruders or those without proper authorization will be stopped. This tunnel represents our Bastion host in the AWS context.

### The AWS Connection

In AWS, this concept translates to:

1. The user accesses our AWS network through an Internet Gateway
2. The Bastion host sits in a public subnet, accessible from the internet
3. The actual resources (like EC2 instances) reside in the private subnet
4. Users must first connect to the Bastion host, and only after authentication can they reach resources in the private subnet

![AWS Bastion Architecture](https://github.com/FeEgyir/Understanding-AWS-Services/blob/3e696426cae9ef536fa5d8ac044efc3f3edc64b5/All%20Images/Bastion%20Host.png)
*Diagram showing public subnet with Bastion host, private subnet with EC2 instance, and user accessing through Internet Gateway*

The Bastion host acts as a secure checkpoint that screens all access attempts. Only after passing through this security layer can users reach the protected resources in the private subnet.

## Implementing a Bastion Host in AWS

I decided to create this entire setup in AWS to see how a Bastion host works in a real environment.

### Step 1: Create a VPC

First, I needed to create a Virtual Private Cloud (VPC), essentially a virtual data center to house all my resources:

1. Navigated to the VPC dashboard in the AWS Console
2. Clicked on "Create VPC"
3. Selected "VPC only" option
4. Entered "test-VPC" as the name
5. Defined an IP range with CIDR block `12.0.0.0/16`
6. Kept the default tags
7. Clicked "Create VPC"

![VPC Creation](images/vpc_creation.png)
*Screenshot showing the VPC creation form with filled details*

After creation, I could see my new VPC alongside the default VPC that AWS provides. For this demonstration, I used my newly created VPC.

### Step 2: Create an Internet Gateway

Next, I created an Internet Gateway to allow users to access my AWS network:

1. From the VPC dashboard, found the Internet Gateway option
2. Clicked "Create internet gateway"
3. Named it "test-IGW"
4. Clicked "Create internet gateway"

Once created, I attached this Internet Gateway to my VPC:

1. Selected the newly created Internet Gateway
2. Clicked "Actions" → "Attach to VPC"
3. Selected "test-VPC"
4. Clicked "Attach internet gateway"

![Internet Gateway Attachment](images/igw_attachment.png)
*Screenshot showing the process of attaching an internet gateway to a VPC*

### Step 3: Create Public and Private Subnets

Now I created two subnets - one public subnet for my Bastion host and one private subnet for my protected resources:

#### Creating the Public Subnet:

1. From the VPC dashboard, navigated to "Subnets"
2. Clicked "Create subnet"
3. Selected "test-VPC"
4. Named the subnet "test-public-subnet-1a" (including the availability zone helps with identification)
5. Selected an Availability Zone (eu-central-1a)
6. Set the CIDR block to `12.0.1.0/24`

#### Creating the Private Subnet:

1. While still in the subnet creation interface, clicked "Add subnet"
2. Named it "test-private-subnet-1a"
3. Selected the same Availability Zone (eu-central-1a)
4. Set the CIDR block to `12.0.3.0/24`
5. Clicked "Create subnet"

![Subnet Creation](images/subnet_creation.png)
*Screenshot of subnet creation form with both subnets configured*

### Step 4: Create Route Tables

I needed to create route tables to control traffic flow between my subnets and the internet:

#### Creating a Route Table for the Public Subnet:

1. Went to "Route Tables" in the VPC dashboard
2. Clicked "Create route table"
3. Named it "test-public-RT" (RT for Route Table)
4. Selected "test-VPC"
5. Kept the default tags
6. Clicked "Create route table"

After creating the route table, I associated it with my public subnet:

1. Selected the newly created route table
2. Went to the "Subnet Associations" tab
3. Clicked "Edit subnet associations"
4. Selected my public subnet
5. Clicked "Save associations"

Next, I added a route to allow internet access:

1. Went to the "Routes" tab
2. Clicked "Edit routes"
3. Clicked "Add route"
4. For the destination, entered `0.0.0.0/0` (meaning all traffic)
5. For the target, selected "Internet Gateway" and chose test-IGW
6. Clicked "Save changes"

![Public Route Table Configuration](images/public_rt_config.png)
*Screenshot showing route table with internet gateway route added*

#### Creating a Route Table for the Private Subnet:

1. Went back to "Route Tables"
2. Clicked "Create route table" 
3. Named it "test-private-RT"
4. Selected "test-VPC"
5. Clicked "Create route table"

After creating the private route table, I associated it with my private subnet:

1. Selected the new private route table
2. Went to "Subnet Associations"
3. Clicked "Edit subnet associations"
4. Selected my private subnet
5. Clicked "Save associations"

For the private subnet, I didn't need to add any additional routes, as the default local route allowed communication within the VPC but prevented direct internet access - exactly what I wanted for my private resources.

At this point, I had set up the networking foundation for my Bastion host architecture:

1. Created a VPC
2. Set up an Internet Gateway and attached it to my VPC
3. Created public and private subnets
4. Configured route tables for both subnets, with internet access for the public subnet only

### Step 5: Create the Bastion Host Instance

Now I created the EC2 instance that would serve as my Bastion host in the public subnet:

1. Navigated to EC2 in the AWS Console
2. Clicked "Launch instance"
3. Entered "test-ec2-public-instance-bastion" as the name
4. Chose Ubuntu as the Amazon Machine Image (AMI)
5. Selected t2.micro for the instance type (free tier eligible)
6. Created a key pair for SSH access:
   
   ```
   # Creating a new key pair
   - Clicked "Create new key pair"
   - Entered "keys-for-bastion-host-demo" as the name
   - Downloaded the .pem file (private key) and kept it secure
   - The public key was automatically associated with the instance
   ```

7. Configured network settings:
   - Clicked "Edit" in the Network Settings section
   - Selected "test-VPC"
   - Selected my public subnet
   - Enabled "Auto-assign public IP" (crucial for internet access)
   - Created a security group named "test-ec2-public-instance-SG"
   - Added a rule for SSH (Port 22) with source "Anywhere" (0.0.0.0/0)
     - Note: In production, I would restrict this to my specific IP for better security

8. Left the storage settings at their defaults
9. Clicked "Launch instance"

![Bastion Host EC2 Creation](images/bastion_ec2_creation.png)
*Screenshot of EC2 creation page with Bastion host settings*

### Step 6: Create the Private Instance

Next, I created an EC2 instance in the private subnet to represent my protected resource:

1. Went back to EC2 dashboard and clicked "Launch instance" again
2. Entered "test-ec2-private-instance" as the name
3. Selected the same Ubuntu AMI
4. Chose t2.micro for the instance type
5. Used the same key pair as before
6. Configured network settings:
   - Selected "test-VPC"
   - Selected my private subnet
   - Disabled "Auto-assign public IP" (important since this instance shouldn't be directly accessible from the internet)
   - Created a new security group: "test-ec2-private-instance-SG"
   - Added an SSH rule (Port 22) with source of my public subnet CIDR range (12.0.1.0/24)
     - This critical configuration means only traffic from my public subnet (where the Bastion host is) can connect to this instance

7. Kept the default storage settings
8. Clicked "Launch instance"

![Private EC2 Creation](images/private_ec2_creation.png)
*Screenshot of EC2 creation page with private instance settings*

After a few moments, both instances were in the "running" state, which I verified in the EC2 dashboard.

## Testing the Bastion Host Setup

Now came the moment of truth - testing if I could access the private instance through the Bastion host!

### Step 7: Connect to the Bastion Host

First, I connected to the Bastion host:

1. From the EC2 dashboard, selected the Bastion host instance
2. Clicked "Connect"
3. Selected the "SSH client" tab to see the connection instructions

Following these steps in my terminal:

```bash
# Change the permissions of the private key file
chmod 400 path/to/keys-for-bastion-host-demo.pem

# Connect to the Bastion host using SSH
ssh -i "path/to/keys-for-bastion-host-demo.pem" ubuntu@bastion-public-ip
```

![SSH to Bastion](images/ssh_to_bastion.png)
*Terminal screenshot showing successful connection to Bastion host*

### Step 8: Copy the Private Key to the Bastion Host

To connect from the Bastion host to the private instance, I needed my private key on the Bastion host:

1. Opened a new terminal window on my local machine
2. Displayed the content of my private key file:
   ```bash
   cat path/to/keys-for-bastion-host-demo.pem
   ```
3. Copied the entire output
4. Switched back to the terminal connected to the Bastion host
5. Created a new file for the key:
   ```bash
   vi aws-ec2-key
   ```
6. Pasted the content, saved and exited (pressed ESC, then typed `:wq`)
7. Set the correct permissions for the key file:
   ```bash
   chmod 400 aws-ec2-key
   ```

> **Note:** In production environments, this approach of copying the private key is not recommended for security reasons. A better practice would be to use SSH key forwarding or AWS Systems Manager Session Manager.

### Step 9: Connect from the Bastion Host to the Private Instance

Now I connected from the Bastion host to the private instance:

1. Got the private IP address of my private instance from the AWS console
2. From the Bastion host terminal, used SSH to connect to the private instance:
   ```bash
   ssh -i aws-ec2-key ubuntu@private-instance-ip
   ```
3. Accepted the connection prompt

To my satisfaction, I was now connected to the private EC2 instance through the Bastion host!

![SSH to Private Instance](images/ssh_to_private_instance.png)
*Terminal screenshot showing successful connection from Bastion to private instance*

## Success! The Bastion Host Architecture is Working

I verified that:

1. I could connect to the Bastion host from my local machine using SSH
2. I could then connect from the Bastion host to the private instance
3. The private instance could not be accessed directly from the internet

This confirmed that my Bastion host was working as intended - providing a secure gateway to access protected resources in my private subnet.

## Summary

In this project, I:

1. Explained the concept of a Bastion host using a castle and tunnel analogy
2. Created a complete AWS network architecture:
   - VPC with CIDR block 12.0.0.0/16
   - Internet Gateway attached to the VPC
   - Public subnet (12.0.1.0/24) with route to the Internet Gateway
   - Private subnet (12.0.3.0/24) without direct internet access
   - Appropriate route tables for both subnets
3. Set up EC2 instances:
   - A Bastion host in the public subnet with internet access
   - A protected instance in the private subnet
4. Configured security properly:
   - Security group for the Bastion host allowing SSH from anywhere
   - Security group for the private instance allowing SSH only from the public subnet
5. Successfully tested the setup by connecting through the Bastion host to the private instance

This Bastion host pattern is a fundamental security architecture in AWS. It provides a single, controlled point of entry to private resources, reducing the attack surface and improving security. By following these steps, I've implemented a secure access mechanism that follows AWS best practices for network security.

## Next Steps for Enhancement

To further improve this setup, I could consider:

- Implementing additional security measures on the Bastion host:
  - Fail2ban to prevent brute force attacks
  - Multi-factor authentication for SSH access
  - IP whitelisting at the security group level
- Setting up SSH key forwarding to avoid storing private keys on the Bastion host
- Using AWS Systems Manager Session Manager as an alternative to a traditional Bastion host
- Setting up CloudWatch alerts for login attempts and suspicious activities
- Implementing automatic shutdown of the Bastion host when not in use to reduce costs
- Creating an Auto Scaling Group for the Bastion host to improve availability
- Setting up a more restrictive Network ACL for the public subnet containing the Bastion host

## Troubleshooting Tips

If you encounter issues with this setup, check the following:

1. Security groups - ensure they allow traffic as expected:
   - The Bastion host security group should allow inbound SSH from your IP
   - The private instance security group should allow inbound SSH only from the public subnet CIDR
   
2. Route tables - verify they're properly associated with the correct subnets:
   - Public subnet route table should have a route to the Internet Gateway
   - Check subnet associations to confirm each subnet uses the correct route table
   
3. Network ACLs - confirm they're not blocking necessary traffic:
   - Default Network ACLs allow all traffic, but custom ACLs might be restrictive
   - Check both inbound and outbound rules
   
4. Key permissions - make sure private key files have 400 permissions:
   ```bash
   chmod 400 path/to/your-key.pem
   ```
   
5. VPC configuration - double-check that the VPC and subnet CIDR blocks don't overlap:
   - Use a tool like a CIDR calculator to verify proper subnet ranges
   - Ensure you can account for all IP addresses within your VPC
   
6. EC2 instance status - confirm both instances are in the "running" state
   
7. Connectivity verification - use ping or telnet to test connectivity at different points:
   ```bash
   # From Bastion host to private instance
   ping private-instance-ip
   
   # Test SSH connectivity
   telnet private-instance-ip 22
   ```

---

This setup demonstrates how to implement the security principle of defense in depth by creating multiple layers of security controls. The Bastion host pattern is just one of many security patterns available in AWS, but it's an important one to understand and implement correctly.
