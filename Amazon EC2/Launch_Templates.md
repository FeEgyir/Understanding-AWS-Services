# AWS EC2 Deployment with Launch Templates

After working with EC2 instances for a while, I found myself repeating the same configuration steps over and over. That's when I discovered AWS EC2 Launch Templates – a game-changer for my workflow. In this post, I'll share how I've been using launch templates to streamline my EC2 deployments.

## Why I Started Using Launch Templates

Before using launch templates, I had to manually configure each EC2 instance with:
- Instance name
- AMI selection
- Instance type
- Key pair
- Security groups
- Storage configuration
- Network settings
- User data scripts for bootstrapping

This process was not only time-consuming but also prone to errors. Launch templates solved this problem by allowing me to save my configurations and reuse them whenever needed.

![EC2 Launch Template Concept](placeholder-image)

## Creating My First Launch Template

Here's how I created my first EC2 launch template with pre-installed Apache and Docker:

### Step 1: Navigating to Launch Templates

From the EC2 dashboard, I clicked on "Launch Templates" in the left sidebar menu. Since I hadn't created any templates before, I clicked the "Create launch template" button.

### Step 2: Configuring Basic Details

I started by providing the following information:
- **Template name**: `ec2-instance-with-apache`
- **Description**: `This template installs and configures Apache web server on Amazon Linux`
- **Template tags**: Added a tag with key `Name` and value `EC2-with-Apache`

### Step 3: Selecting AMI and Instance Type

For the AMI, I clicked "Browse more AMIs" and selected Ubuntu (free tier eligible) with x86 architecture. For the instance type, I chose t2.micro to stay within the free tier.

### Step 4: Key Pair Configuration

I created a new key pair by clicking "Create new key pair" and named it "SSH". The private key automatically downloaded to my computer.

### Step 5: Network Settings

For the security groups, I selected two pre-existing security groups:
- One allowing SSH access (port 22)
- Another allowing HTTP access (port 80)

I had previously created these security groups in the VPC section of AWS. The SSH security group allows me to connect to the instance, while the HTTP security group allows web traffic to reach the Apache server.

### Step 6: Storage Configuration

I kept the default storage configuration with an 8GB EBS volume.

### Step 7: Adding User Data Script

In the "Advanced details" section, I added the following user data script to install both Apache and Docker:

```bash
#!/bin/bash
# Update package repositories
yum update -y

# Install Apache web server
yum install -y httpd

# Create a simple web page
echo "<html><body><h1>Hello World from EC2 User Data!</h1><p>This server was configured automatically using EC2 user data.</p></body></html>" > /var/www/html/index.html

# Start and enable Apache service
systemctl start httpd
systemctl enable httpd
```

### Step 8: Creating the Template

Finally, I clicked "Create launch template" and received confirmation that my template was created successfully.

## Launching an Instance from My Template

With my template ready, launching an instance became much simpler:

1. From the EC2 dashboard, I clicked the dropdown next to "Launch instance" and selected "Launch instance from template"
2. Selected my newly created template from the dropdown
3. Verified that all settings were pre-filled correctly
4. Clicked "Launch instance"

Within minutes, my instance was up and running. I verified that everything worked by:

1. Checking that Apache was installed by opening the instance's public IP in my browser
2. Connecting to the instance via SSH and  confirming the Apache installation

```bash
# Command to SSH into the instance
ssh -i "my-key.pem" ec2-user@ec2-instance-public-ip

# Verifying Apache service status
sudo systemctl status httpd

# Checking Apache version
httpd -v

# Examining the default web page
cat /var/www/html/index.html

# Confirming Apache is set to start on boot
sudo systemctl is-enabled httpd
```

## Creating a Template from an Existing Template

What I found particularly useful was the ability to create new templates based on existing ones. I wanted to create a template that only installed Apache without any other services, so:

1. I navigated to the "Launch Templates" section again
2. Clicked "Create launch template"
3. Named it "ec2-instance-with-apache-only"
4. Under "Source template," I selected my previous template "ec2-instance-with-apache"
5. AWS pre-filled all the settings from the source template
6. I modified the user data script to keep only the essential Apache installation:

```bash
#!/bin/bash
# Update package repositories
yum update -y
# Install Apache web server
yum install -y httpd
# Start and enable Apache service
systemctl start httpd
systemctl enable httpd
```

7. Clicked "Create launch template"

This way, I created a new template by modifying just what I needed while keeping all other settings the same.

## Testing My Second Template

To verify my second template worked correctly:

1. I launched a new instance using the template with only Apache
2. Confirmed Apache was working by accessing the instance's public IP
3. SSH'd into the instance and verified Docker was NOT installed:

```bash
# Connect to instance
ssh -i "my-key.pem" ec2-user@ec2-instance-public-ip

# Check Apache status
systemctl status httpd

# Verify no other services were installed
docker --version
```

As expected, I received a "command not found" error for Docker, confirming that my template only installed Apache as intended.

## Benefits I've Discovered

After using launch templates for a while, I've found several advantages:

1. **Consistency**: My instances are configured exactly the same way every time
2. **Efficiency**: Launching new instances takes seconds instead of minutes
3. **Versioning**: I can create different versions of my templates for different purposes
4. **Reusability**: I can create new templates based on existing ones
5. **Reduced errors**: No more forgetting to configure security groups or other settings

## Security Best Practices I Learned

Working with launch templates taught me some important security practices:

1. **Keep private keys secure**: After downloading the .pem file, I made sure to `chmod 400` it to restrict permissions
2. **Use specific security groups**: I created dedicated security groups rather than opening all ports
3. **Minimize installed software**: I created different templates for different purposes rather than installing everything on every instance

## Conclusion

EC2 Launch Templates have dramatically improved my workflow when working with AWS. Instead of repeatedly configuring the same settings, I now have templates ready for different scenarios. This not only saves me time but also ensures consistency across my infrastructure.

Next, I plan to explore how to use these templates with Auto Scaling groups to automatically scale my applications based on demand. I'm also looking into incorporating more sophisticated user data scripts to handle more complex deployments.

If you're still manually configuring each EC2 instance, I highly recommend giving launch templates a try. The time savings alone are worth it!

---

**Additional Resources I Found Helpful:**
- [AWS EC2 Launch Templates Documentation](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-launch-templates.html)
- [Auto Scaling with Launch Templates](https://docs.aws.amazon.com/autoscaling/ec2/userguide/create-launch-template.html)
- [EC2 Instance Types](https://aws.amazon.com/ec2/instance-types/)
