# AWS EC2 Instances: My Learning Notes

## Introduction
In this chapter of my AWS learning journey, I'm exploring AWS EC2 instances, one of the most fundamental building blocks when working with AWS. Having already covered the sign-up process, IAM user creation, and IAM roles and policies in the previous chapter, I'm now moving to EC2 as the next logical step in learning AWS.

## What I'll Cover in These Notes
In this section, I'll document:
- What EC2 instances are
- How to launch an EC2 instance
- Creating and managing public and private keys
- Associating keys with an EC2 instance
- SSHing from my local machine terminal to a remote EC2 instance running on AWS

This is a basic chapter to help me understand EC2 instances before moving to more complex topics. In my future learning, I'll increase the complexity level and explore setting up EC2 instances with proper VPCs, subnets, and other advanced configurations. For now, I'm focusing on the essential elements to get comfortable with EC2 instances.

## Launching an EC2 Instance

### Navigating to EC2 Dashboard
To start working with EC2 instances, I went to my AWS console and typed "EC2" in the search box. Clicking on EC2 took me to the EC2 dashboard where I could see all running instances. In my personal AWS account, I had zero instances running at this point.
![image alt](https://github.com/FeEgyir/Understanding-AWS-Services/blob/9c4efd25ecfb095f9c29cd07242587a1dfaf9d16/Amazon%20EC2/Launch%20and%20SSH%20into%20an%20EC2%20Instance/images/Instance.png)

### Creating a New Instance
From the EC2 dashboard, I found the orange "Launch instance" button and clicked on it. Now I needed to specify several parameters essential to start an EC2 instance:

1. **Named my instance**: I entered "Test Demo EC2 Instance" for this example.

2. **Selected an Amazon Machine Image (AMI)**:
   I learned that EC2 instances are essentially virtual machines or virtual servers running on AWS. I needed to choose which base operating system I wanted to use. There were multiple options available:
   - Amazon Linux
   - macOS
   - Ubuntu
   - Windows
   - Red Hat
   - SUSE Linux
   - And many more

   Since I'm learning AWS and noticed that 90% of cases use Linux servers, I chose Ubuntu, which is commonly used across the industry. Amazon Linux is also quite popular, but as a beginner, I decided to go with a Linux distribution I'm familiar with.

   I selected "Canonical Ubuntu 22.04 LTS" instance. As I'm just learning, I stuck with the free tier option to avoid unnecessary costs.

3. **Chose Instance Type**:
   I selected t2.micro, which is a very small instance that's free under the AWS free tier. I noticed I could choose larger instances like t2.small or t2.large with higher CPU and memory, but for learning purposes, t2.micro is sufficient.

4. **Created a Key Pair**:
   This was a crucial step. I learned that once I launch an EC2 instance, I need to be able to SSH or remote login to that instance, and without a key pair, this wouldn't be possible.

   I clicked on "Create key pair" and entered "test-demo-key-pair" as the name. I kept the default options (RSA and .pem file) and clicked "Create key pair". This downloaded a .pem file to my computer - "test-demo-key.pem". This file is my private key which I need to keep secure.

   The concept of key pairs is interesting: When creating a key pair, AWS generates two keys - a public key and a private key. The public key is stored with the EC2 instance, while the private key is downloaded to my computer. I'll use this private key later to SSH into the instance.

5. **Set Network Settings**:
   For this basic session on EC2, I kept everything default. I didn't create any custom VPC, subnet, or security group. 
   The only thing I needed to do was check "Allow SSH traffic" and select "Anywhere" from the dropdown. This means I'm allowing remote SSH login to this instance from any IP address in the world (0.0.0.0/0).

   I noticed I could select "My IP" instead, which would only allow SSH access from my current IP address. For demo purposes, I kept it as "Anywhere", though for production environments, I'd restrict this access.

7. **Configured Storage**:
   I kept the default 8GB since this was just for learning purposes.

8. **Advanced Details**:
   I didn't change anything here for this basic example.

Once I filled in all the required information, I clicked "Launch instance". It took just a moment to launch.

## Accessing EC2 Instance Details

After launching, I could see the instance ID. I clicked on this ID to go to the EC2 instance dashboard. I noted that to get back to this dashboard in the future, I could click on "EC2" in the search box, and then click on "Instances running: 1" to see all my instances.

When I clicked on my instance ID, I found all the details about my instance:
- Public IP address
- Private IP address
- Instance state (showing "running")
- Platform (Ubuntu)
- Operating system (Linux)
- Version (Ubuntu Linux 22.04)
- Key pair name that I associated with it

I found this dashboard to be very useful for troubleshooting and investigating EC2 instances.
![image alt](https://github.com/FeEgyir/Understanding-AWS-Services/blob/85d59cc1886aa22f079f4f3d0529fc40977e7a59/Amazon%20EC2/Launch%20and%20SSH%20into%20an%20EC2%20Instance/images/lanuched.png)

## SSHing into My EC2 Instance

To SSH into my EC2 instance, I clicked on the instance ID and found the "Connect" option. I clicked on "Connect" and selected the "SSH client" tab.

I followed these instructions:

1. First, I needed to ensure I have the my private key file that I downloaded earlier. I opened a terminal and navigated to where my file existed:

   ```bash
   ls -lt
   ```

2. Next, I copied the SSH command from the AWS console, which looked like:

   ```bash
   ssh -i "test-demo-key.pem" ubuntu@ec2-xx-xx-xx-xx.compute-1.amazonaws.com
   ```

   Breaking down this command:
   - `ssh` is the command
   - `-i` specifies the private key file
   - `ubuntu` is the default user created when spinning up an Ubuntu EC2 instance
   - The last part is the public DNS name of the instance

   I noted that instead of the public DNS, I could also use the public IPv4 address of the instance - both would work fine.

3. I ran the SSH command in my terminal. When prompted "Are you sure you want to connect?", I typed "yes".

I was now logged into my AWS EC2 instance! To verify I was in the right place, I checked:

- The IP address information shown when I logged in
- Compared it with the private IP in my AWS console to confirm I was in the correct EC2 instance
- I also ran `uname -r` to verify the kernel version

## Terminating My EC2 Instance

After I was done exploring the instance, I terminated it:

1. I typed `exit` to leave the SSH session and return to my local system
2. Went back to the EC2 dashboard
3. Selected my instance
4. Went to "Action" > "Instance State" > "Terminate instance"

I noticed there was also an option to "Stop instance" instead. I learned that stopping preserves the instance so I can restart it later, while terminating permanently destroys it.

For this learning exercise, I chose "Terminate instance" and confirmed. It took a couple of minutes to terminate completely.

## Conclusion

This was my first hands-on experience with EC2 instances. In my future learning, I plan to dive deeper into EC2 instances, exploring what I can do with them in professional environments, what precautions I should take, and what best practices I should follow.

This basic knowledge will serve as a foundation as I continue my AWS learning journey. Next, I'll explore more advanced EC2 concepts and configurations.
