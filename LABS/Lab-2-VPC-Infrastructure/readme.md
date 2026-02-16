# Lab 2: Building Your Amazon VPC Infrastructure

## Lab Overview
**Welcome to your second hands-on lab!**

In this lab, we're going to build a complete VPC infrastructure from scratch. This is where everything we learned in Module 3 comes together in a practical, hands-on way.

As an AWS solutions architect, understanding VPC networking is crucial. Today, you'll create your own Virtual Private Cloud with public and private subnets, set up internet access, configure security, and launch EC2 instances in both environments.

**What You'll Build:**

```
Internet
    |
Internet Gateway
    |
VPC (10.0.0.0/16)
    |
    ├── Public Subnet (10.0.0.0/24)
    │   ├── Public EC2 Instance
    │   └── NAT Gateway
    │
    └── Private Subnet (10.0.2.0/23)
        └── Private EC2 Instance
```

**Lab Objectives:**

By the end of this lab, you'll know how to:
- Create an Amazon VPC with proper CIDR planning
- Set up public and private subnets in different scenarios
- Configure internet gateways and route tables
- Create and configure security groups for different use cases
- Launch EC2 instances in public and private subnets
- Set up NAT gateways for private subnet internet access
- Use AWS Systems Manager Session Manager for secure connections
- Test connectivity between public and private resources

---

## Prerequisites

**Before You Start:**

- Basic understanding of IP addressing and CIDR notation
- Familiarity with AWS Management Console
- Completion of Module 3: Networking concepts

**What AWS Provides:**
- A clean AWS account environment
- Necessary IAM permissions
- Session Manager access for EC2 connections

---

## Task 1: Create Your VPC Foundation

**Let's start by building the main VPC that will house all our resources.**

### Step 1.1: Access VPC Console

1. **Open the AWS Management Console**
2. **Search for "VPC"** in the top search bar
3. **Click on "VPC"** to open the VPC dashboard

### Step 1.2: Create the Main VPC

1. **In the left navigation, click "Your VPCs"**
2. **Click "Create VPC"**
3. **Configure your VPC:**
   - **Resources to create:** Select "VPC only"
   - **Name tag:** Enter `Lab VPC`
   - **IPv4 CIDR:** Enter `10.0.0.0/16`
   - **IPv6 CIDR block:** Leave as "No IPv6 CIDR block"
   - **Tenancy:** Default

4. **Click "Create VPC"**

**What Just Happened?**
You created a VPC with 65,536 IP addresses (10.0.0.0 to 10.0.255.255). This gives you plenty of room to create multiple subnets.

### Step 1.3: Enable DNS Hostnames

1. **Select your "Lab VPC"**
2. **Click "Actions" → "Edit VPC settings"**
3. **Check "Enable DNS hostnames"**
4. **Click "Save"**

**Why This Matters:**
This setting gives your EC2 instances friendly DNS names like `ec2-52-42-133-255.us-west-2.compute.amazonaws.com` instead of just IP addresses.

---

## Task 2: Create Subnets for Organization

**Now let's divide our VPC into public and private areas.**

### Step 2.1: Create the Public Subnet

1. **In the left navigation, click "Subnets"**
2. **Click "Create subnet"**
3. **Configure your public subnet:**
   - **VPC ID:** Select "Lab VPC"
   - **Subnet name:** Enter `Public Subnet`
   - **Availability Zone:** Choose the first AZ in the list
   - **IPv4 subnet CIDR block:** Enter `10.0.0.0/24`

4. **Click "Create subnet"**

### Step 2.2: Configure Public IP Assignment

1. **Select your "Public Subnet"**
2. **Click "Actions" → "Edit subnet settings"**
3. **Check "Enable auto-assign public IPv4 address"**
4. **Click "Save"**

**Understanding the CIDR:**
- VPC: 10.0.0.0/16 = 65,536 addresses
- Public Subnet: 10.0.0.0/24 = 256 addresses (10.0.0.0 to 10.0.0.255)

### Step 2.3: Create the Private Subnet

1. **Click "Create subnet"**
2. **Configure your private subnet:**
   - **VPC ID:** Select "Lab VPC"
   - **Subnet name:** Enter `Private Subnet`
   - **Availability Zone:** Choose the same AZ as your public subnet
   - **IPv4 subnet CIDR block:** Enter `10.0.2.0/23`

3. **Click "Create subnet"**

**Why /23 for Private?**
The /23 gives us 512 addresses (10.0.2.0 to 10.0.3.255). Private subnets usually need more IPs since most resources should be private.

---

## Task 3: Create Internet Gateway

**Let's connect our VPC to the internet.**

### Step 3.1: Create the Gateway

1. **In the left navigation, click "Internet gateways"**
2. **Click "Create internet gateway"**
3. **Name tag:** Enter `Lab IGW`
4. **Click "Create internet gateway"**

### Step 3.2: Attach to VPC

1. **Click "Actions" → "Attach to VPC"**
2. **Select "Lab VPC"** from the dropdown
3. **Click "Attach internet gateway"**

**What's an Internet Gateway?**
Think of it as the front door to your VPC. It allows traffic to flow between your VPC and the internet, and it handles the NAT translation for public IP addresses.

---

## Task 4: Set Up Route Tables

**Now we need to tell traffic how to get around our network.**

### Step 4.1: Create Public Route Table

1. **In the left navigation, click "Route tables"**
2. **Click "Create route table"**
3. **Configure:**
   - **Name:** Enter `Public Route Table`
   - **VPC:** Select "Lab VPC"
4. **Click "Create route table"**

### Step 4.2: Add Internet Route

1. **Select your "Public Route Table"**
2. **Click the "Routes" tab**
3. **Click "Edit routes"**
4. **Click "Add route"**
5. **Configure the new route:**
   - **Destination:** Enter `0.0.0.0/0` (this means "everywhere")
   - **Target:** Select "Internet Gateway" → choose your Lab IGW
6. **Click "Save changes"**

### Step 4.3: Associate with Public Subnet

1. **Click the "Subnet associations" tab**
2. **Click "Edit subnet associations"**
3. **Select "Public Subnet"**
4. **Click "Save associations"**

**Route Table Logic:**
- Local route: 10.0.0.0/16 → Local (automatic)
- Internet route: 0.0.0.0/0 → Internet Gateway

This makes the public subnet truly "public" because it has a path to the internet.

---

## Task 5: Create Security Groups

**Let's set up our firewall rules.**

### Step 5.1: Create Public Security Group

1. **In the left navigation, click "Security groups"**
2. **Click "Create security group"**
3. **Configure:**
   - **Security group name:** Enter `Public SG`
   - **Description:** Enter `Allows HTTP traffic to public instances`
   - **VPC:** Select "Lab VPC"

4. **In Inbound rules, click "Add rule":**
   - **Type:** HTTP
   - **Source:** Anywhere-IPv4 (0.0.0.0/0)

5. **Add a tag:**
   - **Key:** Name
   - **Value:** Public SG

6. **Click "Create security group"**

### Step 5.2: Create Private Security Group

1. **Click "Create security group"**
2. **Configure:**
   - **Security group name:** Enter `Private SG`
   - **Description:** Enter `Allows HTTP from public security group`
   - **VPC:** Select "Lab VPC"

3. **In Inbound rules, click "Add rule":**
   - **Type:** HTTP
   - **Source:** Custom
   - **In the source box, type "sg" and select "Public SG"**

4. **Add a tag:**
   - **Key:** Name
   - **Value:** Private SG

5. **Click "Create security group"**

**Security Group Strategy:**
- Public SG: Allows HTTP from anywhere (internet users)
- Private SG: Only allows HTTP from Public SG (layered security)

---

## Task 6: Launch Public EC2 Instance

**Let's put a web server in our public subnet.**

### Step 6.1: Start Instance Creation

1. **Search for "EC2" in the top search bar**
2. **Click "EC2" to open the EC2 dashboard**
3. **Click "Launch instances"**

### Step 6.2: Configure Basic Settings

1. **Name:** Enter `Public Instance`
2. **Application and OS Images:** Keep "Amazon Linux" and "Amazon Linux 2023 AMI"
3. **Instance type:** Select `t3.micro`
4. **Key pair:** Choose "Proceed without a key pair"

### Step 6.3: Network Configuration

1. **Click "Edit" in Network settings**
2. **Configure:**
   - **VPC:** Lab VPC
   - **Subnet:** Public Subnet
   - **Auto-assign public IP:** Enable
   - **Firewall:** Select existing security group
   - **Security group:** Public SG

### Step 6.4: Advanced Configuration

1. **Expand "Advanced details"**
2. **IAM instance profile:** Select the role with "EC2InstProfile" in the name
3. **User data:** Paste this script:

```bash
#!/bin/bash
# Install Apache web server
yum update -y
yum install -y httpd php8.1
systemctl enable httpd.service
systemctl start httpd
cd /var/www/html
wget https://us-west-2-tcprod.s3.amazonaws.com/courses/ILT-TF-200-ARCHIT/v7.10.1.prod-74eebdd0/lab-2-VPC/scripts/instanceData.zip
unzip instanceData.zip
```

4. **Click "Launch instance"**

**What the Script Does:**
- Updates the system
- Installs Apache web server and PHP
- Downloads and sets up a sample website
- Starts the web server automatically

### Step 6.5: Verify Instance

1. **Click "View all instances"**
2. **Wait for "Instance state" to show "Running"**
3. **Wait for "Status check" to show "2/2 checks passed"**

---

## Task 7: Test Public Instance Access

**Let's verify our public instance is accessible from the internet.**

### Step 7.1: Test Web Access

1. **Select your "Public Instance"**
2. **In the "Details" tab, find "Public IPv4 DNS"**
3. **Copy the DNS name** (don't click the "open address" link)
4. **Open a new browser tab**
5. **Paste the DNS name in the address bar**

**Expected Result:**
You should see a webpage displaying the instance ID and availability zone. This proves your public instance is reachable from the internet!

### Step 7.2: Test Session Manager Access

1. **Go back to EC2 console**
2. **Select "Public Instance" and click "Connect"**
3. **Choose "Session Manager" tab**
4. **Click "Connect"**

5. **In the terminal, run:**
```bash
cd ~
curl -I https://aws.amazon.com/training/
```

**Expected Result:**
You should see HTTP headers from AWS, proving your instance can reach the internet.

---

## Task 8: Create NAT Gateway for Private Access

**Now let's give our private subnet internet access without making it public.**

### Step 8.1: Create NAT Gateway

1. **Go back to VPC console**
2. **In the left navigation, click "NAT gateways"**
3. **Click "Create NAT gateway"**
4. **Configure:**
   - **Name:** Enter `Lab NGW`
   - **Subnet:** Public Subnet (important!)
   - **Elastic IP allocation ID:** Click "Allocate Elastic IP"
5. **Click "Create NAT gateway"**

### Step 8.2: Create Private Route Table

1. **In the left navigation, click "Route tables"**
2. **Click "Create route table"**
3. **Configure:**
   - **Name:** Enter `Private Route Table`
   - **VPC:** Lab VPC
4. **Click "Create route table"**

### Step 8.3: Add NAT Route

1. **Select "Private Route Table"**
2. **Click "Routes" tab → "Edit routes"**
3. **Click "Add route":**
   - **Destination:** `0.0.0.0/0`
   - **Target:** NAT Gateway → select your Lab NGW
4. **Click "Save changes"**

### Step 8.4: Associate with Private Subnet

1. **Click "Subnet associations" tab**
2. **Click "Edit subnet associations"**
3. **Select "Private Subnet"**
4. **Click "Save associations"**

**NAT Gateway Concept:**
The NAT gateway sits in the public subnet and allows private instances to reach the internet for updates and downloads, but prevents internet users from reaching the private instances.

---

## Task 9: Launch Private EC2 Instance

**Let's create a private instance that can't be reached from the internet.**

### Step 9.1: Launch Private Instance

1. **Go to EC2 console → "Launch instances"**
2. **Configure:**
   - **Name:** `Private Instance`
   - **AMI:** Amazon Linux 2023 AMI
   - **Instance type:** t3.micro
   - **Key pair:** Proceed without a key pair

### Step 9.2: Network Configuration

1. **Edit Network settings:**
   - **VPC:** Lab VPC
   - **Subnet:** Private Subnet
   - **Auto-assign public IP:** Disable
   - **Security group:** Private SG

### Step 9.3: Advanced Configuration

1. **IAM instance profile:** Select EC2InstProfile role
2. **User data:** Same script as before:

```bash
#!/bin/bash
yum update -y
yum install -y httpd php8.1
systemctl enable httpd.service
systemctl start httpd
cd /var/www/html
wget https://us-west-2-tcprod.s3.amazonaws.com/courses/ILT-TF-200-ARCHIT/v7.10.1.prod-74eebdd0/lab-2-VPC/scripts/instanceData.zip
unzip instanceData.zip
```

3. **Click "Launch instance"**

---

## Task 10: Test Private Instance Connectivity

**Let's verify our private instance works correctly.**

### Step 10.1: Connect via Session Manager

1. **Select "Private Instance" → "Connect"**
2. **Choose "Session Manager" tab**
3. **Click "Connect"**

4. **Test internet connectivity:**
```bash
cd ~
curl -I https://aws.amazon.com/training/
```

**Expected Result:**
The private instance can reach the internet through the NAT gateway!

### Step 10.2: Test Inter-Instance Communication

1. **Connect to "Public Instance" via Session Manager**
2. **Get the private IP of your private instance** from EC2 console
3. **From the public instance, run:**
```bash
curl [PRIVATE-IP-ADDRESS]
```

Replace [PRIVATE-IP-ADDRESS] with the actual private IP (like 10.0.2.15).

**Expected Result:**
You should see the HTML of the private instance's webpage, proving the public instance can communicate with the private instance.

---

## Task 11: Optional - Test Network Connectivity

**If you have extra time, let's test ICMP connectivity.**

### Step 11.1: Add ICMP Rule

1. **Go to EC2 → Security Groups**
2. **Select "Private SG"**
3. **Click "Inbound rules" → "Edit inbound rules"**
4. **Add rule:**
   - **Type:** Custom ICMP - IPv4
   - **Source:** Custom → select "Public SG"
5. **Click "Save rules"**

### Step 11.2: Test Ping

1. **Connect to Public Instance via Session Manager**
2. **Ping the private instance:**
```bash
ping [PRIVATE-IP-ADDRESS]
```

**Expected Result:**
The ping should now work, showing network connectivity between the instances.

---

## Lab Summary

**Congratulations! You've built a complete VPC infrastructure.**

**What You Created:**
- ✅ VPC with proper CIDR planning (10.0.0.0/16)
- ✅ Public subnet with internet access (10.0.0.0/24)
- ✅ Private subnet with outbound-only internet access (10.0.2.0/23)
- ✅ Internet Gateway for public connectivity
- ✅ NAT Gateway for private outbound access
- ✅ Route tables directing traffic appropriately
- ✅ Security groups implementing layered security
- ✅ EC2 instances in both public and private subnets
- ✅ Session Manager access for secure administration

**Architecture Benefits:**
- **Security:** Private resources protected from direct internet access
- **Scalability:** Room for growth with large CIDR blocks
- **Reliability:** Resources distributed across availability zones
- **Manageability:** Clear separation between public and private resources

**Real-World Applications:**
This architecture is perfect for:
- Web applications with databases
- Multi-tier applications
- Microservices architectures
- Any setup requiring both public-facing and protected resources

**Key Concepts Reinforced:**
- CIDR notation and IP planning
- Route tables and traffic flow
- Security groups vs NACLs
- Public vs private subnet differences
- NAT gateway functionality
- Internet gateway operations

---

## Clean-Up Instructions

**To avoid charges, clean up your resources:**

1. **Terminate EC2 Instances:**
   - Select both instances → Instance state → Terminate instance

2. **Delete NAT Gateway:**
   - VPC → NAT Gateways → Select Lab NGW → Actions → Delete NAT Gateway

3. **Release Elastic IP:**
   - EC2 → Elastic IPs → Select the IP → Actions → Release Elastic IP Address

4. **Delete VPC:**
   - VPC → Your VPCs → Select Lab VPC → Actions → Delete VPC
   - This will delete all associated subnets, route tables, and security groups

**Note:** The internet gateway will be automatically detached and deleted when you delete the VPC.

---

## Troubleshooting Guide

**Common Issues:**

**Problem:** Can't reach public instance webpage
- **Check:** Security group allows HTTP (port 80)
- **Check:** Instance has public IP address
- **Check:** Route table has internet gateway route

**Problem:** Private instance can't reach internet
- **Check:** NAT gateway is in public subnet
- **Check:** Private route table points to NAT gateway
- **Check:** NAT gateway has Elastic IP

**Problem:** Session Manager won't connect
- **Check:** IAM instance profile is attached
- **Check:** Instance is running and passed status checks
- **Wait:** Session Manager can take a few minutes to become available

**Problem:** Ping between instances fails
- **Check:** Security groups allow ICMP traffic
- **Check:** Using correct private IP addresses
- **Check:** Instances are in same VPC

---

## Next Steps

**After completing this lab:**

1. **Experiment:** Try adding more subnets in different AZs
2. **Scale:** Launch additional instances and test load balancing
3. **Secure:** Explore NACLs for additional network security
4. **Connect:** Learn about VPC peering and Transit Gateways
5. **Monitor:** Set up CloudWatch for network monitoring

**Prepare for:** Module 4 where we'll dive deeper into compute services and see how they integrate with the VPC infrastructure you just built!

Great job completing this comprehensive VPC lab! 🎉