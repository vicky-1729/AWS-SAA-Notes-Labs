4# Lab: Complete Amazon S3 Hands-On Workshop

## Lab Overview

In this comprehensive lab, you'll learn Amazon S3 from the ground up by building a complete solution that demonstrates all key S3 features. You'll create buckets, upload objects, configure security, set up static website hosting, implement lifecycle policies, and explore different storage classes.

**Estimated Time:** 90-120 minutes  
**Difficulty Level:** Beginner to Intermediate  
**Prerequisites:** AWS Account with appropriate permissions

---

## What You'll Build

By the end of this lab, you'll have:
- Multiple S3 buckets with different configurations
- A static website hosted on S3
- Lifecycle policies for cost optimization
- Security configurations including encryption and access control
- Experience with different storage classes
- Versioning and backup strategies

---

## Lab Architecture

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Website       │    │   Media         │    │   Archive       │
│   Bucket        │    │   Bucket        │    │   Bucket        │
│                 │    │                 │    │                 │
│ - Static Site   │    │ - Images        │    │ - Backups       │
│ - Public Read   │    │ - Videos        │    │ - Lifecycle     │
│ - Versioning    │    │ - Intelligent   │    │ - Glacier       │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

---

## Lab Sections

1. **S3 Basics** - Creating buckets and uploading objects
2. **Security Configuration** - Access control and encryption
3. **Static Website Hosting** - Host a complete website
4. **Storage Classes** - Optimize costs with different tiers
5. **Lifecycle Management** - Automate transitions
6. **Versioning & Backup** - Data protection strategies
7. **Advanced Features** - Event notifications and more

---

## Section 1: S3 Basics

### Step 1.1: Create Your First S3 Bucket

1. **Open AWS Management Console**
   - Navigate to the S3 service
   - Click on "Create bucket"

2. **Configure Basic Settings**
   ```
   Bucket name: your-name-website-lab-2024
   Region: US East (N. Virginia) us-east-1
   ```
   > **Note:** Bucket names must be globally unique. Replace "your-name" with your actual name or initials.

3. **Keep Default Settings**
   - Leave "Block all public access" checked for now
   - Keep versioning disabled for now
   - Click "Create bucket"

### Step 1.2: Create Additional Buckets

Create two more buckets for different purposes:

1. **Media Bucket**
   ```
   Bucket name: your-name-media-lab-2024
   Region: US East (N. Virginia) us-east-1
   ```

2. **Archive Bucket**
   ```
   Bucket name: your-name-archive-lab-2024
   Region: US East (N. Virginia) us-east-1
   ```

### Step 1.3: Upload Your First Objects

1. **Navigate to your website bucket**
   - Click on the bucket name to open it

2. **Create folder structure**
   - Click "Create folder"
   - Create folders: `images`, `css`, `js`

3. **Upload test files**
   - Create a simple HTML file on your computer:
   
   ```html
   <!DOCTYPE html>
   <html>
   <head>
       <title>My S3 Website</title>
   </head>
   <body>
       <h1>Welcome to My S3 Website</h1>
       <p>This website is hosted on Amazon S3!</p>
   </body>
   </html>
   ```
   - Save it as `index.html`
   - Upload it to your bucket root

4. **Upload additional content**
   - Find any image file (jpg, png) and upload to `images/` folder
   - Create a simple CSS file and upload to `css/` folder

---

## Section 2: Security Configuration

### Step 2.1: Configure Bucket Encryption

1. **Enable Default Encryption**
   - Go to your website bucket
   - Click on "Properties" tab
   - Scroll down to "Default encryption"
   - Click "Edit"

2. **Configure SSE-S3 Encryption**
   ```
   Server-side encryption: Enable
   Encryption key type: Amazon S3 managed keys (SSE-S3)
   ```
   - Click "Save changes"

### Step 2.2: Test Private Access

1. **Try to access your HTML file**
   - Click on `index.html` in your bucket
   - Click on "Object URL" 
   - Try to open the URL in a new browser tab
   - **Expected Result:** Access Denied error

### Step 2.3: Create IAM Policy for S3 Access

1. **Go to IAM Service**
   - Create a new policy with this JSON:

   ```json
   {
       "Version": "2012-10-17",
       "Statement": [
           {
               "Effect": "Allow",
               "Action": [
                   "s3:GetObject",
                   "s3:PutObject",
                   "s3:DeleteObject"
               ],
               "Resource": "arn:aws:s3:::your-name-website-lab-2024/*"
           },
           {
               "Effect": "Allow",
               "Action": [
                   "s3:ListBucket"
               ],
               "Resource": "arn:aws:s3:::your-name-website-lab-2024"
           }
       ]
   }
   ```
   - Replace bucket name with your actual bucket name
   - Save policy as "S3-Website-Access"

---

## Section 3: Static Website Hosting

### Step 3.1: Enable Static Website Hosting

1. **Configure Website Hosting**
   - Go to your website bucket
   - Click "Properties" tab
   - Scroll to "Static website hosting"
   - Click "Edit"

2. **Set Website Configuration**
   ```
   Static website hosting: Enable
   Hosting type: Host a static website
   Index document: index.html
   Error document: error.html
   ```

3. **Create Error Page**
   - Create an `error.html` file:
   ```html
   <!DOCTYPE html>
   <html>
   <head>
       <title>Page Not Found</title>
   </head>
   <body>
       <h1>404 - Page Not Found</h1>
       <p>Sorry, the page you're looking for doesn't exist.</p>
   </body>
   </html>
   ```
   - Upload to bucket root

### Step 3.2: Configure Public Access

1. **Modify Block Public Access**
   - Go to "Permissions" tab
   - Click "Edit" on "Block public access"
   - **Uncheck all boxes**
   - Type "confirm" and save

2. **Create Bucket Policy**
   - Still in "Permissions" tab
   - Click "Edit" on "Bucket policy"
   - Add this policy:

   ```json
   {
       "Version": "2012-10-17",
       "Statement": [
           {
               "Sid": "PublicReadGetObject",
               "Effect": "Allow",
               "Principal": "*",
               "Action": "s3:GetObject",
               "Resource": "arn:aws:s3:::your-name-website-lab-2024/*"
           }
       ]
   }
   ```
   - Replace bucket name with yours
   - Click "Save changes"

### Step 3.3: Test Your Website

1. **Get Website URL**
   - Go to "Properties" tab
   - Find "Static website hosting" section
   - Copy the "Bucket website endpoint" URL

2. **Test the Website**
   - Open URL in browser
   - Verify your HTML page loads
   - Test error handling by adding `/nonexistent` to URL

### Step 3.4: Enhance Your Website

1. **Create Better HTML**
   - Replace `index.html` with:

   ```html
   <!DOCTYPE html>
   <html>
   <head>
       <title>My S3 Portfolio</title>
       <link rel="stylesheet" href="css/style.css">
   </head>
   <body>
       <header>
           <h1>Welcome to My Portfolio</h1>
           <nav>
               <a href="#about">About</a>
               <a href="#projects">Projects</a>
               <a href="#contact">Contact</a>
           </nav>
       </header>
       
       <main>
           <section id="about">
               <h2>About Me</h2>
               <p>This is my portfolio website hosted on Amazon S3!</p>
               <img src="images/profile.jpg" alt="Profile" width="200">
           </section>
           
           <section id="projects">
               <h2>My Projects</h2>
               <div class="project">
                   <h3>S3 Static Website</h3>
                   <p>A complete website hosted on Amazon S3 with lifecycle policies.</p>
               </div>
           </section>
           
           <section id="contact">
               <h2>Contact</h2>
               <p>Email: your-email@example.com</p>
           </section>
       </main>
       
       <footer>
           <p>&copy; 2024 My Portfolio. Hosted on Amazon S3.</p>
       </footer>
   </body>
   </html>
   ```

2. **Create CSS File**
   - Create `css/style.css`:

   ```css
   body {
       font-family: Arial, sans-serif;
       margin: 0;
       padding: 20px;
       background-color: #f5f5f5;
   }
   
   header {
       background-color: #232f3e;
       color: white;
       padding: 20px;
       text-align: center;
   }
   
   nav a {
       color: white;
       text-decoration: none;
       margin: 0 15px;
   }
   
   main {
       max-width: 800px;
       margin: 20px auto;
       background: white;
       padding: 20px;
       border-radius: 5px;
   }
   
   section {
       margin-bottom: 30px;
   }
   
   .project {
       border: 1px solid #ddd;
       padding: 15px;
       border-radius: 5px;
       margin: 10px 0;
   }
   
   footer {
       text-align: center;
       padding: 20px;
       background-color: #232f3e;
       color: white;
   }
   ```

3. **Upload files and test**
   - Upload both files
   - Refresh your website to see the changes

---

## Section 4: Storage Classes and Optimization

### Step 4.1: Explore Storage Classes

1. **Upload Test Files with Different Storage Classes**
   - Upload a large file (>5MB) to your media bucket
   - During upload, change storage class to "Standard-IA"
   - Upload another file with "Intelligent-Tiering"

2. **Compare Storage Classes**
   - View objects and note storage class differences
   - Check the "Properties" of each object

### Step 4.2: Configure Intelligent Tiering

1. **Enable Intelligent Tiering**
   - Go to media bucket
   - Click "Management" tab
   - Click "Create Intelligent-Tiering configuration"

2. **Set Configuration**
   ```
   Configuration name: MediaOptimization
   Status: Enabled
   Scope: Apply to all objects
   ```

### Step 4.3: Test Different Access Patterns

1. **Create Test Scenario**
   - Upload several files to test different scenarios
   - Some files you'll access frequently
   - Others you'll rarely access

---

## Section 5: Lifecycle Management

### Step 5.1: Create Your First Lifecycle Rule

1. **Navigate to Management Tab**
   - Go to your archive bucket
   - Click "Management" tab
   - Click "Create lifecycle rule"

2. **Configure Basic Rule**
   ```
   Lifecycle rule name: ArchivePolicy
   Status: Enabled
   Rule scope: Apply to all objects in the bucket
   ```

### Step 5.2: Set Transition Actions

1. **Configure Transitions**
   ```
   Current version transitions:
   - After 30 days: Transition to Standard-IA
   - After 90 days: Transition to Glacier Instant Retrieval
   - After 365 days: Transition to Glacier Deep Archive
   ```

2. **Configure Expiration (Optional)**
   ```
   Delete expired delete markers: Enabled
   Delete incomplete multipart uploads: After 7 days
   ```

### Step 5.3: Test Lifecycle Rules

1. **Upload Test Files**
   - Upload several files with different dates (if possible)
   - Or upload files and modify their metadata to simulate age

2. **Monitor Transitions**
   - Check back over time to see transitions
   - Note: Actual transitions take 24-48 hours

---

## Section 6: Versioning and Backup

### Step 6.1: Enable Versioning

1. **Enable Versioning on Website Bucket**
   - Go to website bucket
   - Click "Properties" tab
   - Find "Bucket Versioning"
   - Click "Edit" and select "Enable"

### Step 6.2: Test Versioning

1. **Modify and Re-upload File**
   - Edit your `index.html` file
   - Add a new section or change content
   - Upload the modified file with the same name

2. **View Version History**
   - In the bucket, click "Show versions"
   - See multiple versions of `index.html`
   - Click on each version to see differences

### Step 6.3: Restore Previous Version

1. **Practice Version Restoration**
   - Download an older version
   - Make it the current version by re-uploading
   - Verify the website shows the older content

### Step 6.4: Lifecycle Rules for Versions

1. **Create Version-Aware Lifecycle Rule**
   ```
   Rule name: VersionCleanup
   Scope: All objects
   
   Noncurrent version transitions:
   - After 30 days: Move to Standard-IA
   - After 90 days: Move to Glacier
   
   Noncurrent version expiration:
   - Delete after 365 days
   ```

---

## Section 7: Advanced Features

### Step 7.1: Cross-Region Replication (Optional)

1. **Create Destination Bucket**
   - Create a bucket in a different region
   - Example: `your-name-backup-west-2024` in us-west-2

2. **Set Up Replication**
   - Go to source bucket "Management" tab
   - Click "Create replication rule"
   - Configure cross-region replication

### Step 7.2: Event Notifications

1. **Create SNS Topic (Optional)**
   - Go to SNS service
   - Create a topic for S3 notifications

2. **Configure S3 Event Notifications**
   - In bucket "Properties"
   - Find "Event notifications"
   - Create notification for object creation

### Step 7.3: S3 Access Logging

1. **Enable Access Logging**
   - Create a separate bucket for logs
   - Enable access logging on main bucket
   - Configure log delivery

---

## Section 8: Monitoring and Analytics

### Step 8.1: CloudWatch Metrics

1. **View S3 CloudWatch Metrics**
   - Go to CloudWatch console
   - Find S3 metrics
   - Observe bucket size, request metrics

2. **Set Up Alarms (Optional)**
   - Create alarm for bucket size
   - Create alarm for request errors

### Step 8.2: S3 Storage Class Analysis

1. **Enable Storage Class Analysis**
   - Go to bucket "Metrics" tab
   - Create storage class analysis
   - Wait for data to accumulate

---

## Section 9: Cost Optimization

### Step 9.1: Review Cost Implications

1. **Estimate Costs**
   - Use AWS Pricing Calculator for S3
   - Compare costs of different storage classes
   - Calculate potential savings with lifecycle policies

2. **Implement Cost Controls**
   - Set up billing alerts
   - Review S3 usage in Cost Explorer

---

## Section 10: Security Best Practices

### Step 10.1: Implement Additional Security

1. **Enable MFA Delete**
   - Enable MFA delete on versioned bucket (CLI required)

2. **Review Access Patterns**
   - Use CloudTrail to monitor S3 access
   - Review IAM policies for least privilege

### Step 10.2: Data Protection

1. **Test Backup and Recovery**
   - Simulate accidental deletion
   - Practice recovery procedures

---

## Lab Cleanup

### Important: Clean Up Resources

1. **Empty All Buckets**
   - Select all objects and versions
   - Delete permanently

2. **Delete Buckets**
   - Delete all created buckets
   - Verify no charges will continue

3. **Remove IAM Policies**
   - Delete custom IAM policies created
   - Remove any test users/roles

---

## Key Takeaways

### What You've Learned

✅ **S3 Fundamentals**
- Created and configured S3 buckets
- Understood object storage concepts
- Managed bucket and object permissions

✅ **Static Website Hosting**
- Hosted a complete website on S3
- Configured public access securely
- Implemented error handling

✅ **Storage Optimization**
- Used different storage classes
- Implemented lifecycle policies
- Configured intelligent tiering

✅ **Data Protection**
- Enabled versioning for data protection
- Implemented backup strategies
- Understood encryption options

✅ **Cost Management**
- Learned about cost optimization strategies
- Implemented automated transitions
- Monitored usage and costs

### Real-World Applications

- **Static Website Hosting**: Host marketing sites, documentation
- **Backup Storage**: Implement organization-wide backup solutions
- **Data Archiving**: Long-term retention for compliance
- **Content Distribution**: Store and serve media files
- **Data Lakes**: Foundation for big data analytics

### Next Steps

1. **Explore S3 Transfer Acceleration** for global users
2. **Learn about S3 Select** for querying data in place
3. **Study S3 Cross-Region Replication** for disaster recovery
4. **Investigate S3 Batch Operations** for large-scale operations
5. **Connect S3 with other AWS services** (Lambda, CloudFront, etc.)

---

## Troubleshooting Guide

### Common Issues and Solutions

**Issue**: Access Denied when accessing website
- **Solution**: Check bucket policy and public access settings

**Issue**: Lifecycle rules not working
- **Solution**: Wait 24-48 hours for AWS to apply transitions

**Issue**: High costs
- **Solution**: Review storage classes and implement lifecycle policies

**Issue**: Objects not replicating
- **Solution**: Check IAM roles and bucket permissions

**Issue**: Versioning taking too much space
- **Solution**: Implement lifecycle rules for non-current versions

---

## Additional Resources

- [Amazon S3 Documentation](https://docs.aws.amazon.com/s3/)
- [S3 Best Practices](https://docs.aws.amazon.com/s3/latest/userguide/optimizing-performance.html)
- [S3 Pricing Calculator](https://calculator.aws/)
- [AWS Well-Architected Framework](https://aws.amazon.com/architecture/well-architected/)

---

**Congratulations!** You've completed a comprehensive S3 lab covering all major features. This hands-on experience provides a solid foundation for using S3 in real-world scenarios.