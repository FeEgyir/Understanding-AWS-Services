# AWS VPC and EC2 Setup: 

## Introduction

In this part of my AWS learning journey, I'll be setting up EC2 instances in both public and private subnets. By the end of this section, I'll understand several key AWS networking concepts including:

- VPC (Virtual Private Cloud)
- CIDR range creation and assignment
- Public and private subnet configuration
- Internet Gateway and NAT Gateway setup
- Route table creation and association
- EC2 instance provisioning within subnets

To begin, I need an active AWS account. I've already logged into my AWS console homepage.

<!-- IMAGE SUGGESTION: Screenshot of AWS console homepage after logging in -->

## Creating a VPC

The first component I need to create is a VPC (Virtual Private Cloud), which will serve as the network foundation for my resources. For this VPC, I'll use the CIDR range 11.0.0.0/16.

Here's how I created my VPC:

1. In the AWS console search box, I typed "VPC" and clicked on the VPC link
2. From the VPC dashboard, I clicked "Create VPC"
3. AWS offers two options: "VPC only" and "VPC and more"
   - While "VPC and more" would create multiple resources at once, I chose "VPC only" to better understand each component

<!-- IMAGE SUGGESTION: Screenshot of the VPC creation page showing the "VPC only" option selected -->

4. I entered these settings:
   - Name: AWS-demo-VPC
   - IPv4 CIDR block: 11.0.0.0/16
   - Tenancy: Default (kept the default setting)
   - Tags: Used the same name tag that was automatically added
5. Clicked "Create VPC"

My VPC was successfully created.

<!-- IMAGE SUGGESTION: Screenshot showing the successful creation confirmation with VPC details -->

## Understanding CIDR Ranges

Before continuing, I wanted to understand CIDR ranges better. CIDR (Classless Inter-Domain Routing) notation is used to specify IP address ranges.

I used a CIDR calculator (found by searching "CIDR range calculator" on Google) to explore this concept:

1. When I entered my VPC CIDR range (11.0.0.0/16) into the calculator:
   - It showed I would have 65,536 available IP addresses within my VPC
   - The IP range starts at 11.0.0.0 and ends at 11.0.255.255

<!-- IMAGE SUGGESTION: Screenshot of CIDR calculator showing the results for 11.0.0.0/16 -->

2. I also explored how changing the suffix number affects the available IPs:
   - Smaller numbers (like /8) provide more IP addresses
   - Larger numbers (like /32) provide fewer IPs (a /32 gives just one IP address)
   - My choice of /16 for the VPC gives a good balance of available addresses

Understanding CIDR ranges is important because subnets within a VPC need to use portions of the VPC's overall CIDR range.

## Creating Subnets

With my VPC created, I needed to set up two subnets:
- A public subnet for resources that need direct internet access
- A private subnet for resources that should not be directly accessible from the internet

For these subnets, I'll use smaller CIDR blocks:
- Public subnet: 11.0.1.0/24
- Private subnet: 11.0.2.0/24

Using the CIDR calculator, I confirmed that each /24 subnet provides 256 IP addresses (ranging from x.x.x.0 to x.x.x.255).

<!-- IMAGE SUGGESTION: Screenshot of CIDR calculator showing the results for a /24 subnet -->

Here's how I created the subnets:

### Creating the Public Subnet

1. From the VPC dashboard, I clicked on "Subnets" in the left navigation and then "Create subnet"
2. Selected my "AWS-demo-VPC" from the dropdown
3. Entered the following details:
   - Subnet name: AWS-demo-public-subnet
   - Availability Zone: eu-north-1a (I'm working in the Europe region)
   - IPv4 CIDR block: 11.0.1.0/24 (providing 256 IP addresses)
4. I noticed AWS confirmed I would have 256 IPs available in this subnet

<!-- IMAGE SUGGESTION: Screenshot of the public subnet creation form filled out -->

### Creating the Private Subnet

1. While still in the subnet creation flow, I clicked "Add new subnet"
2. Entered the following details:
   - Subnet name: AWS-demo-private-subnet
   - Availability Zone: eu-north-1b (using a different AZ for redundancy)
   - IPv4 CIDR block: 11.0.2.0/24 (providing another 256 IP addresses)
3. Clicked "Create subnet"

Both my subnets were created successfully within my VPC.

<!-- IMAGE SUGGESTION: Screenshot showing both subnets listed in the subnet dashboard -->

## Creating an Internet Gateway

Next, I needed to create an Internet Gateway to allow my public subnet to access the internet. An Internet Gateway is crucial because:

- It allows resources in the public subnet to be accessible from the internet
- It enables two-way communication between resources in the public subnet and the internet
- It acts as a bridge between my VPC and the wider internet

Here's how I created it:

1. From the VPC dashboard, I clicked on "Internet Gateways" in the left navigation
2. Clicked "Create internet gateway"
3. Named it "AWS-demo-IGW" (IGW being short for Internet Gateway)
4. Clicked "Create internet gateway"

<!-- IMAGE SUGGESTION: Screenshot of the Internet Gateway creation page -->

Once created, the Internet Gateway was in a "detached" state. I needed to attach it to my VPC:

1. Selected the newly created Internet Gateway
2. Clicked "Actions" and selected "Attach to VPC"
3. Selected my "AWS-demo-VPC" from the dropdown
4. Clicked "Attach internet gateway"

The Internet Gateway was successfully attached to my VPC.

<!-- IMAGE SUGGESTION: Screenshot showing the attached state of the Internet Gateway -->

## Creating and Configuring Route Tables

Creating an Internet Gateway isn't enough to enable internet access. I also needed to configure route tables to direct traffic properly. Route tables contain rules (routes) that determine where network traffic is directed.

I needed to create two route tables:
- A public route table for the public subnet
- A private route table for the private subnet

### Creating the Route Tables

1. From the VPC dashboard, I clicked on "Route tables" in the left navigation
2. Clicked "Create route table"
3. For the public route table:
   - Named it "AWS-demo-public-route-table"
   - Selected my VPC ("AWS-demo-VPC")
   - Clicked "Create route table"

<!-- IMAGE SUGGESTION: Screenshot of creating the public route table -->

4. Repeated the process for the private route table:
   - Named it "AWS-demo-private-route-table"
   - Selected the same VPC
   - Clicked "Create route table"

### Associating Route Tables with Subnets

With both route tables created, I needed to associate them with their respective subnets:

1. For the public route table:
   - Selected the public route table by clicking on its ID
   - Went to the "Subnet associations" tab
   - Clicked "Edit subnet associations"
   - Selected my public subnet ("AWS-demo-public-subnet")
   - Clicked "Save associations"

<!-- IMAGE SUGGESTION: Screenshot showing the subnet association page for the public route table -->

2. For the private route table:
   - Selected the private route table
   - Went to the "Subnet associations" tab
   - Clicked "Edit subnet associations"
   - Selected my private subnet ("AWS-demo-private-subnet")
   - Clicked "Save associations"

### Adding a Route to the Internet Gateway

To enable internet access for the public subnet, I needed to add a route to the Internet Gateway in the public route table:

1. Selected the public route table
2. Went to the "Routes" tab
3. Clicked "Edit routes"
4. Clicked "Add route"
5. For the new route, entered:
   - Destination: 0.0.0.0/0 (representing all IP addresses, meaning "any destination on the internet")
   - Target: Selected "Internet Gateway" and then chose my Internet Gateway from the dropdown
6. Clicked "Save changes"

<!-- IMAGE SUGGESTION: Screenshot of adding the 0.0.0.0/0 route to the Internet Gateway -->

The destination 0.0.0.0/0 is important to understand - it's a catch-all that means "send any traffic not destined for within the VPC to the Internet Gateway." This effectively enables internet access for resources in the public subnet.

## Understanding the NAT Gateway Configuration

The next component I need to create is a NAT Gateway for my private subnet. A NAT (Network Address Translation) Gateway serves a different purpose than an Internet Gateway:

- Internet Gateway: Enables two-way communication between VPC resources and the internet. Resources in public subnets can both send and receive traffic
- NAT Gateway: Enables one-way communication where resources in private subnets can initiate outbound connections to the internet, but external systems cannot initiate connections to those resources

I found NAT Gateways essential when I needed to update or maintain resources like EC2 instances or RDS databases in private subnets. These resources need internet access to download updates and patches, but I wanted to keep them secure from inbound connections.
<!-- IMAGE SUGGESTION: A conceptual diagram showing the difference between Internet Gateway (two-way) and NAT Gateway (one-way) traffic flows -->

## Creating a NAT Gateway

*[Screenshot opportunity: NAT Gateway section in AWS Console]*

To create a NAT Gateway:

1. I navigated to the AWS console and selected "NAT Gateway" from the left sidebar
2. I clicked on "Create NAT Gateway"
3. For the name, I entered `AWS-Demo-NAT-Gateway`
4. For subnet selection, I chose my public subnet

> **Important:** NAT Gateways must be placed in public subnets to function correctly.

*[Screenshot opportunity: NAT Gateway creation form with subnet selection]*

5. For the Elastic IP allocation, I clicked "Allocate Elastic IP"
   - The system assigned IP `13.50.172.193` to my NAT Gateway

> **Note:** NAT Gateways require an Elastic IP address to function.

6. I skipped the additional settings and clicked "Create NAT Gateway"

*[Screenshot opportunity: NAT Gateway creation confirmation with "pending" status]*

After a few minutes, the status changed from "pending" to "available" with a green indicator.

> **Cost Warning:** Unlike some AWS services, NAT Gateways are not free. I'm careful to delete them when not in use to avoid unnecessary charges.

## Configuring Route Tables for NAT Gateway

*[Screenshot opportunity: Route Tables section in AWS Console]*

With my NAT Gateway ready, I needed to update the private route table:

1. I went back to "Route Tables" in the AWS console
2. I selected my private route table
3. I clicked on "Edit routes"
4. I added a new route with:
   - Destination: `0.0.0.0/0` (all traffic)
   - Target: Selected my newly created NAT Gateway from the dropdown

*[Screenshot opportunity: Edit routes form showing NAT Gateway selection]*

5. I clicked "Save changes"

After saving, I could see the updated route table with the NAT Gateway route in an active status.

## Verifying My Network Configuration

To ensure everything was set up correctly, I reviewed both route tables:

- **Private Route Table:** Now includes a route to the NAT Gateway (`0.0.0.0/0` → NAT Gateway)
- **Public Route Table:** Contains a route to the Internet Gateway (`0.0.0.0/0` → Internet Gateway)

*[Screenshot opportunity: Side-by-side comparison of public and private route tables]*

## Creating and Connecting to EC2 Instances 

After setting up my VPC networking components, I needed to create EC2 instances to test the configuration. I started with an instance for my public subnet.

*[Screenshot opportunity: EC2 dashboard showing "Launch Instance" button]*

1. From the AWS Console homepage, I searched for "EC2" and clicked on the EC2 service
2. I clicked on "Launch Instance" to start the creation process
3. For the instance name, I entered `AWS-Demo-EC2-Public`
4. For the Amazon Machine Image (AMI), I selected Ubuntu
5. I kept the 64-bit architecture and chose t3.micro as my instance type

### Creating an SSH Key Pair

*[Screenshot opportunity: Key pair creation dialog box]*

When creating EC2 instances, I needed an SSH key pair to securely connect:

1. I clicked "Create key pair"
2. I named it `AWS-Demo-SSH-Key-For-Public-EC2`
3. I clicked "Create key pair" and the private key file automatically downloaded to my local machine
4. The public key was automatically associated with my EC2 instance

> **Note:** The private key file is critical for SSH access. I moved this file to a dedicated folder (`AWS-Demo`) for safekeeping.

### Configuring Network Settings

*[Screenshot opportunity: Network settings configuration panel]*

For the networking configuration:

1. I clicked "Edit" in the Network Settings section
2. I selected my demo VPC (`11.0.0.0/16`)
3. I chose my public subnet
4. I enabled "Auto-assign public IP" to ensure the instance would be accessible from the internet
5. For security groups, I added two rules:
   - SSH (port 22) - to allow remote access via SSH
   - ICMP - to allow ping from anywhere (`0.0.0.0/0`)

6. I kept the default storage (8GB)
7. I clicked "Launch instance"

*[Screenshot opportunity: EC2 launch confirmation screen]*

## Creating the Private EC2 Instance

Next, I created a second EC2 instance for my private subnet.

1. I clicked "Launch Instance" again
2. I named this instance `AWS-Demo-EC2-Private`
3. I selected Ubuntu as the AMI and t3.micro for the instance type

### Creating Another SSH Key Pair

1. I created another key pair named `AWS-Demo-SSH-Key-For-Private-EC2`
2. The private key downloaded automatically, and I moved it to my `AWS-Demo` folder

### Configuring Private Network Settings

*[Screenshot opportunity: Private EC2 network settings configuration]*

For the networking configuration:

1. I selected my demo VPC
2. I chose my private subnet
3. I disabled "Auto-assign public IP" since this is a private instance
4. For security groups, I added two rules:
   - SSH (port 22) - for SSH access
   - ICMP - but this time I restricted the source to my public subnet CIDR (`11.0.1.0/24`)

> **Important:** I can't access the private instance directly from the internet, but I can access it from resources within my public subnet. This is why I specified the public subnet CIDR (`11.0.1.0/24`) as the source for the ICMP rule.

5. I kept the default 8GB storage
6. I clicked "Launch instance"

## Connecting to the Public EC2 Instance

*[Screenshot opportunity: EC2 Connect panel showing SSH instructions]*

After both instances were running (showing a green "Running" status):

1. I selected my public EC2 instance
2. I clicked "Connect"
3. I selected the "SSH client" tab to view connection instructions
4. In my local terminal, I navigated to my `AWS-Demo` directory where I stored my key files
5. I ran `ls -l` to verify the keys were present

### Setting Proper Key Permissions

For security reasons, the SSH key files need restrictive permissions:

```bash
chmod 400 AWS-Demo-SSH-Key-For-Public-EC2.pem
chmod 400 AWS-Demo-SSH-Key-For-Private-EC2.pem
```

I ran these commands to make both key files read-only.

### Establishing SSH Connection

I copied the SSH command from the AWS console:

```bash
ssh -i "AWS-Demo-SSH-Key-For-Public-EC2.pem" ubuntu@ec2-xx-xx-xx-xx.compute-1.amazonaws.com
```

After running this command:
1. I accepted the security prompt by typing "yes"
2. I successfully connected to my public EC2 instance

*[Screenshot opportunity: Terminal showing successful SSH connection]*

## Testing Connectivity

To verify my public instance was properly configured:

1. I confirmed the private IP address was `11.0.1.250`, matching what I saw in the AWS console
2. From a new terminal window on my local machine, I pinged the public IP address of my EC2 instance
3. The ping was successful, confirming my public instance was accessible from the internet

# Testing Private EC2 Instance Connectivity

To fully verify my network setup, I needed to test internet connectivity from both my public and private EC2 instances.

First, I confirmed internet access from my public EC2 instance:

```bash
curl ipinfo.io
```

*[Screenshot opportunity: Terminal showing successful curl response from public EC2]*

The command returned all the IP information details, confirming that my public EC2 instance had proper internet connectivity through the Internet Gateway.

## Testing Access to Private EC2 from Outside

Next, I wanted to demonstrate that the private EC2 instance cannot be accessed directly from the internet.

*[Screenshot opportunity: Diagram showing access pattern - external machine → public EC2 → private EC2]*

From my local machine's terminal, I attempted to ping the private EC2 instance using its private IP address:

```bash
ping 11.0.2.104
```

As expected, all requests timed out. This confirms that my private EC2 instance is not directly accessible from outside the VPC, which is exactly what I wanted for security purposes.

## Accessing Private EC2 from Public EC2

However, I confirmed that the private EC2 instance is accessible from within the VPC by pinging it from my public EC2 instance:

```bash
ping 11.0.2.104
```

*[Screenshot opportunity: Terminal showing successful ping from public EC2 to private EC2]*

The ping was successful, confirming connectivity between my public and private subnets within the VPC.

## SSH Access to Private EC2 via Public EC2

To access my private EC2 instance for administration, I needed to:

1. First SSH into my public EC2 instance (already completed)
2. Then SSH from the public instance to the private instance

This is a common pattern known as a "bastion host" or "jump server" approach.

### Transferring the Private Key

To SSH from the public instance to the private instance, I needed the private key on my public EC2:

1. On my public EC2 instance, I created a new private key file:

```bash
touch AWS-Demo-SSH-Key-For-Private-EC2.pem
```

2. I opened the file for editing:

```bash
nano AWS-Demo-SSH-Key-For-Private-EC2.pem
```

3. I copied the entire private key content from my local machine and pasted it into the file on the public EC2 instance
   
   > **Note:** When copying SSH keys, ensure there are no extra spaces, line breaks, or missing characters.

4. I set the correct permissions on the key file:

```bash
chmod 400 AWS-Demo-SSH-Key-For-Private-EC2.pem
```

*[Screenshot opportunity: Terminal showing the key file being created and permission set]*

### Connecting to Private EC2

With the key in place, I used the following command to connect to my private EC2 instance:

```bash
ssh -i AWS-Demo-SSH-Key-For-Private-EC2.pem ubuntu@11.0.2.104
```

After accepting the security prompt, I successfully connected to my private EC2 instance.

*[Screenshot opportunity: Terminal showing successful SSH connection to private EC2]*

## Testing Internet Access from Private EC2

The final test was to verify that my private EC2 instance could access the internet through the NAT Gateway:

```bash
curl ipinfo.io
```

*[Screenshot opportunity: Terminal showing successful curl response from private EC2]*

The command returned the IP information details, confirming that my private EC2 instance had outbound internet connectivity through the NAT Gateway.

## How the NAT Gateway Works in This Scenario

When my private EC2 instance makes an internet request:

1. The request first goes to the private subnet's route table
2. The route table has a rule that sends all internet traffic (0.0.0.0/0) to the NAT Gateway
3. The NAT Gateway, which sits in the public subnet, forwards the request to the internet
4. When the response comes back, the NAT Gateway remembers which private instance made the request and routes the response back accordingly

This setup provides the security benefit of keeping my private instances isolated from direct internet access while still allowing them to download updates and patches when needed.

## Conclusion

I've now successfully set up a complete AWS networking environment with:

1. A VPC with public and private subnets
2. Internet Gateway for public subnet internet access
3. NAT Gateway for private subnet outbound internet access
4. Route tables correctly configured for both subnets
5. EC2 instances in both subnets with appropriate security groups
6. Verified connectivity according to our design requirements

This architecture provides a secure foundation for deploying applications where some components need to be publicly accessible (like web servers in the public subnet) while keeping sensitive components (like databases in the private subnet) protected from direct internet access.

*[Screenshot opportunity: AWS console showing the complete architecture with all components]*


## Key Concepts Learned So Far

1. **VPC**: A logically isolated virtual network where I can launch AWS resources.
2. **CIDR Range**: A method to allocate IP addresses, where smaller suffix numbers (/8, /16) provide more IP addresses than larger ones (/24, /32).
3. **Subnets**: Subdivisions of a VPC's IP address range where I can place resources:
   - Public subnets have routes to the internet via an Internet Gateway
   - Private subnets don't have direct internet access
4. **Internet Gateway**: Allows two-way communication between resources in a VPC and the internet.
5. **Route Tables**: Control the traffic flow between subnets and gateways by defining routes based on destination IP addresses.
6. **NAT Gateway**: Allows resources in private subnets to access the internet but prevents the internet from initiating connections to those resources.

