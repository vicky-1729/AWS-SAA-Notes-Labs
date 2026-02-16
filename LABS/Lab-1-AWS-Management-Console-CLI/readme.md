# Lab 1: Exploring and Interacting with the AWS Management Console and AWS CLI

## Lab Overview

Welcome to our first hands-on lab! In this lab, you'll get familiar with the two main ways to interact with AWS:
- **AWS Management Console** (Web-based interface)
- **AWS CLI** (Command Line Interface)

**What you'll learn:**
- Navigate the AWS Management Console
- Create AWS resources using the web interface
- Use AWS CLI to manage resources
- Understand how both methods work with the same AWS API

**Prerequisites:**
- Your own AWS account (Free Tier is sufficient)
- Basic computer skills
- Internet connection

---

## 🚀 Getting Started

### Step 1: Sign in to AWS Console

1. Go to [aws.amazon.com](https://aws.amazon.com)
2. Click **Sign In to the Console**
3. Enter your AWS account credentials
4. You'll land on the AWS Management Console home page

### Step 2: Choose Your AWS Region

**Important:** Always work in a region close to you for better performance.

1. Look at the top-right corner of the console
2. Click on the region dropdown (shows something like "US East (N. Virginia)")
3. Choose a region close to your location:
   - **US East (N. Virginia)** - for East Coast US
   - **US West (Oregon)** - for West Coast US  
   - **Europe (Ireland)** - for Europe
   - **Asia Pacific (Mumbai)** - for India
   - **Asia Pacific (Singapore)** - for Southeast Asia

---

## 🖥️ Task 1: Explore the AWS Management Console

### 1.1: Learn the Console Layout

**Navigation Bar:** Top section with AWS logo, search box, and your account info

**Services Menu:** Click "Services" to see all AWS services organized by category

**Search Box:** Type any service name (like "S3" or "EC2") to quickly find it

### 1.2: Add Services to Favorites

1. Click **Services** in the top navigation
2. Find **S3** under "Storage" 
3. Click the **star icon** next to S3 to add it to favorites
4. Do the same for **EC2** (under "Compute")
5. Now you'll see these services pinned at the top for quick access

### 1.3: Customize Your Dashboard

1. On the home page, click **+ Add widgets**
2. Try adding different widgets like:
   - **Cost and Usage**
   - **Recently Visited Services**
   - **AWS Health**
3. Drag widgets around to rearrange them
4. Resize widgets by dragging their corners

---

## 📦 Task 2: Create an S3 Bucket (Using Console)

**What is S3?** Amazon S3 is like a large drive in the cloud where you can store files.

### Step-by-Step Instructions:

1. **Find S3 Service:**
   - Click **Services** → **Storage** → **S3**
   - OR use the search box and type "S3"

2. **Create New Bucket:**
   - Click **Create bucket**
   - **Bucket name:** `my-first-bucket-[your-name]-[random-numbers]`
   - Example: `my-first-bucket-john-12345`
   - **Region:** Keep the region you selected earlier
   - Leave all other settings as default
   - Click **Create bucket**

3. **Success!** You should see your bucket in the S3 dashboard

### 📁 Upload a File to Your Bucket

1. **Click on your bucket name** to open it
2. Click **Upload**
3. Click **Add files**
4. Choose any small file from your computer (like a photo)
5. Click **Upload**
6. **Success!** Your file is now stored in the cloud

---

## 💻 Task 3: Install and Use AWS CLI

### 3.1: Install AWS CLI

**For Windows:**
1. Download AWS CLI from: https://awscli.amazonaws.com/AWSCLIV2.msi
2. Run the installer
3. Open Command Prompt or PowerShell

**For Mac:**
```bash
curl "https://awscli.amazonaws.com/AWSCLIV2.pkg" -o "AWSCLIV2.pkg"
sudo installer -pkg AWSCLIV2.pkg -target /
```

**For Linux:**
```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

### 3.2: Configure AWS CLI

1. **Get Your Access Keys:**
   - In AWS Console, click your name (top-right) → **Security credentials**
   - Scroll to **Access keys** section
   - Click **Create access key**
   - Choose **Command Line Interface (CLI)**
   - Download or copy your keys

2. **Configure CLI:**
   ```bash
   aws configure
   ```
   Enter when prompted:
   - **AWS Access Key ID:** [Your access key]
   - **AWS Secret Access Key:** [Your secret key] 
   - **Default region:** [Same region you chose earlier, like us-east-1]
   - **Output format:** json

### 3.3: Test AWS CLI

**List your S3 buckets:**
```bash
aws s3 ls
```
You should see the bucket you created earlier!

---

## 🔧 Task 4: Create Resources Using AWS CLI

### 4.1: Create Another S3 Bucket via CLI

```bash
aws s3 mb s3://my-cli-bucket-[your-name]-[random-numbers]
```
Example:
```bash
aws s3 mb s3://my-cli-bucket-john-67890
```

### 4.2: List All Your Buckets

```bash
aws s3 ls
```
You should now see both buckets (one created via console, one via CLI)!

### 4.3: Upload File via CLI

**Create a test file:**
```bash
echo "Hello from AWS CLI!" > test-file.txt
```

**Upload to your CLI bucket:**
```bash
aws s3 cp test-file.txt s3://my-cli-bucket-[your-name]-[random-numbers]/
```

**List contents of your bucket:**
```bash
aws s3 ls s3://my-cli-bucket-[your-name]-[random-numbers]/
```

---

## 🎯 Key Takeaways

**Console vs CLI - Same Results:**
- Both methods use the same AWS API behind the scenes
- Console is visual and user-friendly
- CLI is faster for automation and scripting
- Both can accomplish the same tasks

**When to Use Each:**
- **Console:** Learning, one-time setups, visual exploration
- **CLI:** Automation, scripting, repetitive tasks, DevOps

**Important Concepts:**
- **Regions:** Geographic locations where AWS resources live
- **S3 Buckets:** Container for files with globally unique names
- **AWS API:** The underlying system both console and CLI use

---

## 🧹 Cleanup (Important!)

**To avoid charges, delete your test resources:**

### Delete via Console:
1. Go to S3 in the console
2. Select your bucket
3. Click **Empty** to delete all files
4. Then click **Delete** to remove the bucket

### Delete via CLI:
```bash
# Remove files first
aws s3 rm s3://my-cli-bucket-[your-name]-[random-numbers] --recursive

# Remove bucket
aws s3 rb s3://my-cli-bucket-[your-name]-[random-numbers]
```

---

## 🎉 Congratulations!

You've successfully:
- ✅ Navigated the AWS Management Console
- ✅ Created resources using the web interface  
- ✅ Installed and configured AWS CLI
- ✅ Created resources using command line
- ✅ Understood the relationship between console and CLI

**Next Steps:**
- Explore other AWS services in the console
- Practice more CLI commands
- Try automating tasks with CLI scripts

---

## 🆘 Troubleshooting

**Can't sign in to AWS?**
- Check your email/password
- Make sure you're using the right account type (root user vs IAM user)

**CLI not working?**
- Verify installation: `aws --version`
- Check configuration: `aws configure list`
- Ensure access keys are correct

**Bucket name errors?**
- Bucket names must be globally unique
- Use only lowercase letters, numbers, and hyphens
- Try adding more random numbers to your bucket name

**Getting permission errors?**
- Make sure you're using your own AWS account
- Check that your access keys have proper permissions

---

## 📚 Additional Resources

- [AWS CLI User Guide](https://docs.aws.amazon.com/cli/latest/userguide/)
- [S3 Getting Started Guide](https://docs.aws.amazon.com/s3/latest/userguide/GetStartedWithS3.html)
- [AWS Free Tier Details](https://aws.amazon.com/free/)