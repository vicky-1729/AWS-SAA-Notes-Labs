# Lab 3: Creating a Database Layer in Your Amazon VPC Infrastructure

## Lab Overview

**Welcome to your third hands-on lab!**

In this lab, we're building on the VPC infrastructure from Lab 2 to create a secure, multi-tier architecture. You'll add an Amazon Aurora MySQL database cluster and an Application Load Balancer to create a complete web application stack that follows AWS security best practices.

As an AWS solutions architect, understanding how to properly secure and connect databases is crucial. Today, you'll create a managed database, set up load balancing, and test the complete application flow from internet users to your database.

**What You'll Build:**

```
Internet Users
      |
Application Load Balancer (Public Subnets)
      |
EC2 Web Servers (Private Subnets)
      |
Aurora MySQL Database (Private DB Subnets)
```

**Lab Objectives:**

By the end of this lab, you'll know how to:
- Create an Amazon Aurora MySQL database cluster
- Set up proper security groups for database access
- Create and configure Application Load Balancers
- Create target groups and register EC2 instances
- Test load balancer health checks and failover
- Connect web applications to managed databases
- Monitor RDS instance metadata and performance
- (Optional) Create cross-region read replicas

---

## Prerequisites

**Before You Start:**

- **Completion of Lab 2** (VPC with public/private subnets)
- Basic understanding of databases and web applications
- Familiarity with AWS Management Console

**What You'll Need:**
- Your existing Lab VPC from Lab 2
- Two EC2 instances in private subnets (we'll create these)
- Proper IAM permissions for RDS and ELB

**If You Don't Have Lab 2 Resources:**
Don't worry! We'll provide quick setup instructions to create the required infrastructure.

---

## Pre-Lab Setup: Prepare Your Environment

**If you completed Lab 2, you can skip to Task 1. Otherwise, follow these quick setup steps:**

### Setup Step 1: Create Basic VPC Infrastructure

1. **Create VPC:**
   - Name: `Lab VPC`
   - CIDR: `10.0.0.0/16`
   - Enable DNS hostnames

2. **Create Subnets:**
   - Public Subnet 1: `10.0.1.0/24` (AZ a)
   - Public Subnet 2: `10.0.3.0/24` (AZ b)
   - Private Subnet 1: `10.0.0.0/24` (AZ a)
   - Private Subnet 2: `10.0.2.0/24` (AZ b)

3. **Create Internet Gateway:**
   - Attach to Lab VPC

4. **Create Route Tables:**
   - Public route table with 0.0.0.0/0 → Internet Gateway
   - Associate with both public subnets

### Setup Step 2: Create DB Subnet Group

1. **Go to RDS Console → Subnet Groups**
2. **Create DB subnet group:**
   - Name: `lab-db-subnet-group`
   - Description: `Subnet group for Lab database`
   - VPC: Lab VPC
   - Subnets: Select both private subnets
3. **Click "Create"**

**What's a DB Subnet Group?**
This tells RDS which subnets it can use for your database. By using only private subnets, your database won't be accessible from the internet.

---

## Task 1: Create Security Groups for Database Access

**Let's set up proper security layers for our three-tier architecture.**

### Step 1.1: Create Application Load Balancer Security Group

1. **Go to EC2 Console → Security Groups**
2. **Click "Create security group"**
3. **Configure:**
   - **Name:** `ALB-SG`
   - **Description:** `Allows HTTP traffic to load balancer`
   - **VPC:** Lab VPC

4. **Add Inbound Rules:**
   - **Type:** HTTP (80)
   - **Source:** Anywhere-IPv4 (0.0.0.0/0)

5. **Add Tags:**
   - **Key:** Name
   - **Value:** ALB Security Group

6. **Click "Create security group"**

### Step 1.2: Create Web Server Security Group

1. **Create security group:**
   - **Name:** `WebServer-SG`
   - **Description:** `Allows traffic from load balancer to web servers`
   - **VPC:** Lab VPC

2. **Add Inbound Rules:**
   - **Type:** HTTP (80)
   - **Source:** Custom → Select "ALB-SG"

3. **Add Tags and Create**

### Step 1.3: Create Database Security Group

1. **Create security group:**
   - **Name:** `Database-SG` 
   - **Description:** `Allows MySQL traffic from web servers`
   - **VPC:** Lab VPC

2. **Add Inbound Rules:**
   - **Type:** MySQL/Aurora (3306)
   - **Source:** Custom → Select "WebServer-SG"

3. **Add Tags and Create**

**Security Group Strategy:**
- Internet → ALB (port 80)
- ALB → Web Servers (port 80)  
- Web Servers → Database (port 3306)
- Each layer only accepts traffic from the layer above it!

---

## Task 2: Create Amazon Aurora MySQL Database

**Now let's create our managed database cluster.**

### Step 2.1: Start Database Creation

1. **Go to RDS Console**
2. **Click "Create database"**
3. **Choose "Standard create"**

### Step 2.2: Configure Database Engine

1. **Engine Options:**
   - **Engine type:** Aurora (MySQL Compatible)
   - **Edition:** Aurora MySQL
   - **Version:** Keep default (latest)

2. **Templates:**
   - Select **"Dev/Test"** (good for learning, cheaper than Production)

### Step 2.3: Database Configuration

1. **Settings:**
   - **DB cluster identifier:** `aurora-lab-cluster`
   - **Master username:** `admin`
   - **Password management:** Self managed
   - **Master password:** Create a strong password (save it!)
   - **Confirm password:** Enter same password

**Password Requirements:**
- At least 8 characters
- Include uppercase, lowercase, numbers
- No spaces or special characters like @, ", \

### Step 2.4: Instance Configuration

1. **DB instance class:**
   - **Burstable classes:** db.t3.medium
   
2. **Availability & durability:**
   - **Multi-AZ deployment:** Don't create an Aurora Replica
   
**Why Single AZ for This Lab?**
We're focusing on the basic architecture. In production, you'd use Multi-AZ for high availability.

### Step 2.5: Connectivity Settings

1. **Connectivity:**
   - **VPC:** Lab VPC
   - **DB subnet group:** lab-db-subnet-group
   - **Public access:** No (this is crucial for security!)
   - **VPC security group:** Choose existing
   - **Existing VPC security groups:** Database-SG (remove default)

### Step 2.6: Additional Configuration

1. **Expand "Additional configuration"**
2. **Database options:**
   - **Initial database name:** `inventory`
3. **Backup:**
   - **Backup retention period:** 7 days
4. **Monitoring:**
   - **Enhanced monitoring:** Disable (to reduce costs)
5. **Maintenance:**
   - **Auto minor version upgrade:** Disable

### Step 2.7: Create Database

1. **Review your settings**
2. **Click "Create database"**

**Expected Result:**
You'll see "Successfully created database aurora-lab-cluster" message. The database will take 5-10 minutes to become available.

**While We Wait:**
Let's continue with the next task. RDS will finish creating in the background.

---

## Task 3: Launch Web Server EC2 Instances

**Let's create two web servers that will connect to our database.**

### Step 3.1: Create First Web Server

1. **Go to EC2 Console → Launch instances**
2. **Basic Configuration:**
   - **Name:** `WebServer1`
   - **AMI:** Amazon Linux 2023 AMI
   - **Instance type:** t3.micro
   - **Key pair:** Proceed without a key pair

### Step 3.2: Network Settings

1. **Click "Edit" in Network settings**
2. **Configure:**
   - **VPC:** Lab VPC
   - **Subnet:** Private Subnet 1
   - **Auto-assign public IP:** Disable
   - **Security group:** Select existing → WebServer-SG

### Step 3.3: Advanced Configuration

1. **Expand "Advanced details"**
2. **IAM instance profile:** Select the role that allows Session Manager (if available)
3. **User data:** Paste this script:

```bash
#!/bin/bash
# Install web server and PHP
yum update -y
yum install -y httpd php php-mysqli
systemctl enable httpd
systemctl start httpd

# Create inventory application
cd /var/www/html
cat > index.php << 'EOF'
<?php
$servername = "";
$username = "";
$password = "";
$dbname = "inventory";

// Create connection
$conn = new mysqli($servername, $username, $password, $dbname);

// Check connection
if ($conn->connect_error) {
    die("Connection failed: " . $conn->connect_error);
}

// Display instance information
$instance_id = file_get_contents('http://169.254.169.254/latest/meta-data/instance-id');
$availability_zone = file_get_contents('http://169.254.169.254/latest/meta-data/placement/availability-zone');

echo "<h1>Inventory Application</h1>";
echo "<p>Instance ID: $instance_id</p>";
echo "<p>Availability Zone: $availability_zone</p>";

if ($conn->connect_error) {
    echo "<p>Database Status: <span style='color: red;'>Not Connected</span></p>";
    echo "<p>Error: " . $conn->connect_error . "</p>";
} else {
    echo "<p>Database Status: <span style='color: green;'>Connected</span></p>";
    
    // Create table if it doesn't exist
    $sql = "CREATE TABLE IF NOT EXISTS products (
        id INT AUTO_INCREMENT PRIMARY KEY,
        name VARCHAR(255) NOT NULL,
        price DECIMAL(10,2) NOT NULL,
        quantity INT NOT NULL
    )";
    $conn->query($sql);
    
    // Insert sample data if table is empty
    $result = $conn->query("SELECT COUNT(*) as count FROM products");
    $row = $result->fetch_assoc();
    if ($row['count'] == 0) {
        $conn->query("INSERT INTO products (name, price, quantity) VALUES ('Widget A', 19.99, 100)");
        $conn->query("INSERT INTO products (name, price, quantity) VALUES ('Widget B', 29.99, 75)");
        $conn->query("INSERT INTO products (name, price, quantity) VALUES ('Widget C', 39.99, 50)");
    }
    
    // Display products
    $result = $conn->query("SELECT * FROM products");
    if ($result->num_rows > 0) {
        echo "<h2>Product Inventory</h2>";
        echo "<table border='1' style='border-collapse: collapse; width: 100%;'>";
        echo "<tr><th>ID</th><th>Name</th><th>Price</th><th>Quantity</th></tr>";
        while($row = $result->fetch_assoc()) {
            echo "<tr><td>".$row["id"]."</td><td>".$row["name"]."</td><td>$".$row["price"]."</td><td>".$row["quantity"]."</td></tr>";
        }
        echo "</table>";
    }
}

$conn->close();
?>
EOF

# Set permissions
chown apache:apache /var/www/html/index.php
chmod 644 /var/www/html/index.php
```

4. **Click "Launch instance"**

### Step 3.4: Create Second Web Server

1. **Repeat the process for WebServer2:**
   - **Name:** `WebServer2`
   - **Subnet:** Private Subnet 2 (different AZ)
   - **Same configuration as WebServer1**

**What the Script Does:**
- Installs Apache web server and PHP
- Creates a sample inventory application
- Will connect to our Aurora database
- Shows which server is responding (for load balancer testing)

---

## Task 4: Create Application Load Balancer

**Now let's create our load balancer to distribute traffic.**

### Step 4.1: Create Target Group First

1. **Go to EC2 → Target Groups**
2. **Click "Create target group"**
3. **Configure target group:**
   - **Target type:** Instances
   - **Target group name:** `web-servers-tg`
   - **Protocol:** HTTP
   - **Port:** 80
   - **VPC:** Lab VPC
   - **Health check path:** `/` (default)

4. **Click "Next"**

### Step 4.2: Register Targets

1. **Available instances:** Select both WebServer1 and WebServer2
2. **Click "Include as pending below"**
3. **Click "Create target group"**

**What's a Target Group?**
It's a logical grouping of instances that the load balancer can send traffic to. Health checks ensure traffic only goes to healthy instances.

### Step 4.3: Create Load Balancer

1. **Go to EC2 → Load Balancers**
2. **Click "Create load balancer"**
3. **Choose "Application Load Balancer" → Create**

### Step 4.4: Configure Load Balancer

1. **Basic configuration:**
   - **Name:** `web-app-alb`
   - **Scheme:** Internet-facing
   - **IP address type:** IPv4

2. **Network mapping:**
   - **VPC:** Lab VPC
   - **Mappings:** Select both availability zones
   - **Subnets:** Choose both public subnets

3. **Security groups:**
   - **Remove:** default security group
   - **Select:** ALB-SG

4. **Listeners and routing:**
   - **Listener:** HTTP:80
   - **Default action:** Forward to web-servers-tg

5. **Click "Create load balancer"**

**Expected Result:**
Load balancer will be in "Provisioning" state for a few minutes, then change to "Active."

---

## Task 5: Configure Database Connection

**Let's connect our web application to the Aurora database.**

### Step 5.1: Get Database Endpoint

1. **Go to RDS Console → Databases**
2. **Click on "aurora-lab-cluster"**
3. **In "Connectivity & security" tab:**
   - **Copy the "Writer endpoint"** (looks like aurora-lab-cluster.cluster-xxxxx.region.rds.amazonaws.com)
   - **Note the port:** 3306

### Step 5.2: Update Web Server Configuration

1. **Connect to WebServer1 via Session Manager:**
   - EC2 → Instances → Select WebServer1 → Connect
   - Choose "Session Manager" → Connect

2. **Edit the PHP configuration:**
```bash
sudo nano /var/www/html/index.php
```

3. **Update these lines at the top of the file:**
```php
$servername = "YOUR_DATABASE_ENDPOINT_HERE";
$username = "admin";
$password = "YOUR_PASSWORD_HERE";
```

4. **Save and exit:** Ctrl+X, Y, Enter

5. **Restart Apache:**
```bash
sudo systemctl restart httpd
```

### Step 5.3: Repeat for WebServer2

1. **Connect to WebServer2 via Session Manager**
2. **Make the same configuration changes**
3. **Restart Apache**

**Security Note:**
In production, you'd use AWS Secrets Manager or environment variables for database credentials, not hardcoded passwords!

---

## Task 6: Test Your Application

**Let's verify everything works together.**

### Step 6.1: Check Target Group Health

1. **Go to EC2 → Target Groups**
2. **Select "web-servers-tg"**
3. **Click "Targets" tab**
4. **Wait for both instances to show "healthy"**

**If instances are unhealthy:**
- Check security groups allow port 80
- Verify web servers are running
- Check target group health check settings

### Step 6.2: Test Load Balancer

1. **Go to EC2 → Load Balancers**
2. **Select "web-app-alb"**
3. **Copy the "DNS name"**
4. **Open a new browser tab**
5. **Paste the DNS name**

**Expected Result:**
- You should see the inventory application
- It should show the instance ID and availability zone
- Database status should show "Connected"
- You should see a table with sample products

### Step 6.3: Test Load Balancing

1. **Refresh the page several times**
2. **Watch the "Instance ID" change**
3. **This proves the load balancer is distributing traffic between both servers!**

### Step 6.4: Test Database Persistence

1. **Note which products are displayed**
2. **Refresh multiple times and switch between servers**
3. **The same data appears on both servers**
4. **This proves both servers are connecting to the same database!**

---

## Task 7: Monitor Your Database

**Let's explore RDS monitoring and management.**

### Step 7.1: Explore RDS Console

1. **Go to RDS → Databases → aurora-lab-cluster**
2. **Explore the tabs:**

**Connectivity & security:**
- Writer and reader endpoints
- Security groups
- Subnet group information

**Configuration:**
- Instance class
- Engine version
- Parameter groups
- Storage configuration

**Monitoring:**
- CPU utilization
- Database connections
- Read/write IOPS
- Network throughput

### Step 7.2: View Database Performance

1. **Click "Monitoring" tab**
2. **Review key metrics:**
   - **CPU Utilization:** Should be low for our simple app
   - **Database Connections:** Should show active connections
   - **Read IOPS / Write IOPS:** Shows database activity

**Understanding the Metrics:**
- High CPU might indicate complex queries or high traffic
- Many connections could indicate connection pooling issues
- IOPS shows how busy your storage is

---

## Task 8: Optional - Create Cross-Region Read Replica

**If you have extra time, let's add disaster recovery capability.**

### Step 8.1: Create Read Replica

1. **In RDS → Databases**
2. **Select aurora-lab-cluster**
3. **Actions → Create cross-Region read replica**

### Step 8.2: Configure Replica

1. **Destination Region:** Choose a different region
2. **DB instance identifier:** `aurora-lab-replica`
3. **Instance class:** db.t3.medium
4. **Public access:** No
5. **VPC security group:** Use default for now
6. **Click "Create"**

**What's a Read Replica?**
- Copy of your database in another region
- Handles read-only traffic
- Provides disaster recovery
- Can be promoted to master if needed

### Step 8.3: Verify Replica Creation

1. **Switch to the destination region**
2. **Go to RDS → Databases**
3. **You should see your replica being created**

**Note:** Cross-region replicas take 10-15 minutes to create and can incur additional charges.

---

## Lab Summary

**Congratulations! You've built a complete three-tier web application.**

**What You Created:**
- ✅ Aurora MySQL database cluster in private subnets
- ✅ Application Load Balancer in public subnets
- ✅ Two web servers in private subnets
- ✅ Proper security groups for each layer
- ✅ Target groups with health checking
- ✅ Database connectivity from web application
- ✅ (Optional) Cross-region read replica

**Architecture Benefits:**
- **Security:** Database isolated in private subnets
- **High Availability:** Load balancer distributes traffic across AZs
- **Scalability:** Easy to add more web servers to target group
- **Manageability:** Managed database reduces operational overhead
- **Monitoring:** CloudWatch metrics for performance tracking

**Real-World Applications:**
This architecture is perfect for:
- E-commerce websites
- Content management systems
- Web applications with user data
- API backends with persistent storage
- Multi-tenant SaaS applications

**Key Concepts Reinforced:**
- Three-tier architecture (web, app, database)
- Security group layering and least privilege
- Load balancer target groups and health checks
- RDS subnet groups and private database access
- Application Load Balancer vs Network Load Balancer
- Database endpoint configuration
- Cross-region disaster recovery

---

## Clean-Up Instructions

**To avoid charges, clean up your resources in this order:**

### Step 1: Delete Load Balancer Resources
1. **EC2 → Load Balancers → Select web-app-alb → Actions → Delete**
2. **EC2 → Target Groups → Select web-servers-tg → Actions → Delete**

### Step 2: Terminate EC2 Instances
1. **EC2 → Instances → Select both WebServer instances**
2. **Instance state → Terminate instance**

### Step 3: Delete Database
1. **RDS → Databases → Select aurora-lab-cluster**
2. **Actions → Delete**
3. **Uncheck "Create final snapshot"**
4. **Type "delete me" to confirm**
5. **Click "Delete"**

### Step 4: Delete Security Groups
1. **EC2 → Security Groups**
2. **Delete Database-SG, WebServer-SG, ALB-SG** (in that order)

### Step 5: Clean Up DB Subnet Group
1. **RDS → Subnet groups → Select lab-db-subnet-group → Delete**

**Important:** Delete resources in the correct order to avoid dependency errors!

---

## Troubleshooting Guide

**Common Issues:**

**Problem:** Load balancer shows unhealthy targets
- **Check:** Security groups allow HTTP (port 80)
- **Check:** Web servers are running: `sudo systemctl status httpd`
- **Check:** Target group health check path is correct

**Problem:** Web application can't connect to database
- **Check:** Database security group allows port 3306 from WebServer-SG
- **Check:** Database endpoint is correct in PHP file
- **Check:** Database credentials are correct
- **Check:** Database status is "Available"

**Problem:** Can't connect to instances via Session Manager
- **Check:** IAM instance profile is attached
- **Check:** Instances are in private subnets with proper route tables
- **Wait:** Session Manager can take a few minutes after instance launch

**Problem:** Load balancer DNS name doesn't respond
- **Check:** Load balancer is in "Active" state
- **Check:** Security groups allow HTTP from internet (0.0.0.0/0)
- **Check:** Load balancer is in public subnets
- **Check:** At least one target is healthy

**Problem:** Database creation fails
- **Check:** DB subnet group has subnets in at least 2 AZs
- **Check:** Selected subnets are in the correct VPC
- **Check:** Security group exists in the same VPC

---

## Next Steps

**After completing this lab:**

1. **Experiment:** Try adding more web servers to the target group
2. **Secure:** Explore AWS Secrets Manager for database credentials
3. **Scale:** Learn about Aurora Auto Scaling for databases
4. **Monitor:** Set up CloudWatch alarms for database metrics
5. **Optimize:** Explore Aurora Serverless for variable workloads
6. **Backup:** Test database snapshots and point-in-time recovery

**Prepare for:** Module 5 where we'll explore storage services and how they integrate with your multi-tier applications!

**Advanced Challenges:**
- Add SSL/TLS certificates to your load balancer
- Implement database connection pooling
- Set up automated backups and restore procedures
- Create a staging environment in a different VPC

Great job building your first production-ready web application architecture! 🎉