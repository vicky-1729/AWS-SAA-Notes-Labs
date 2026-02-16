# Lab 6: Configure an Amazon CloudFront Distribution with an Amazon S3 Origin

*Estimated Time: 60-75 minutes*

## Welcome to Global Content Delivery! 🌍

Hey there, performance optimizer! Ready to make your content lightning-fast for users around the world? Today we're diving into Amazon CloudFront – AWS's global content delivery network (CDN) that can make your websites and applications perform like they're running locally, no matter where your users are located!

Think of CloudFront as having superpowers for your content. Instead of users downloading files directly from your servers (which might be thousands of miles away), CloudFront places copies of your content in edge locations around the globe. When someone in Tokyo requests a file stored in Virginia, CloudFront serves it from the nearest edge location – dramatically reducing latency and improving user experience.

## What We're Building Today

```
┌─────────────────── GLOBAL CDN ARCHITECTURE ───────────────────┐
│                                                                │
│   🌏 Global Users                                             │
│        │                                                      │
│        ▼                                                      │
│ ┌──────────────────┐                                         │
│ │   CloudFront     │  ◄─── Edge Locations Worldwide         │
│ │  Distribution    │       (200+ locations)                  │
│ │                  │                                         │
│ └─────────┬────────┘                                         │
│           │                                                  │
│           ├─────────────────┐                                │
│           │                 │                                │
│           ▼                 ▼                                │
│    ┌─────────────┐   ┌──────────────┐                       │
│    │ S3 Origin   │   │ ALB Origin   │                       │
│    │ (Static)    │   │ (Dynamic)    │                       │
│    │             │   │              │                       │
│    │ ┌─────────┐ │   │ ┌──────────┐ │                       │
│    │ │  OAC    │ │   │ │   EC2    │ │                       │
│    │ │Secured  │ │   │ │Instance  │ │                       │
│    │ │Content  │ │   │ │  Pool    │ │                       │
│    │ └─────────┘ │   │ └──────────┘ │                       │
│    └─────────────┘   └──────────────┘                       │
│                                                              │
│  📊 Performance Benefits:                                    │
│  • 90%+ faster load times                                   │
│  • Reduced origin server load                               │
│  • Enhanced security with OAC                               │
│  • Global availability                                      │
└──────────────────────────────────────────────────────────────┘
```

This isn't just about speed – it's about creating a bulletproof, scalable content delivery system that provides an amazing user experience worldwide!

## Learning Objectives

By completing this lab, you'll master:

- **CloudFront Fundamentals**: Understanding distributions, origins, and behaviors
- **Origin Access Control (OAC)**: Securing S3 content behind CloudFront
- **Multi-Origin Setup**: Serving different content types from different sources
- **Cache Behaviors**: Optimizing content delivery for different file types
- **Security Best Practices**: Preventing direct access to origin servers
- **Performance Monitoring**: Understanding CloudFront metrics and optimization

## Prerequisites and Setup

### What You'll Need
- AWS Console access with appropriate permissions
- Basic understanding of S3 and web content delivery
- A test image file for uploading and testing
- Understanding of DNS and HTTP concepts

### Important Notes
🌍 **Global Impact**: CloudFront changes can take 5-15 minutes to propagate globally
💰 **Cost Awareness**: CloudFront charges based on data transfer and requests
🔒 **Security Focus**: We'll implement Origin Access Control for maximum security

---

## Task 1: Explore Your Existing CloudFront Distribution

Let's start by understanding what's already built for you. Think of this as getting the keys to a sports car and learning how all the controls work before you start customizing it.

### 1.1 Navigate to CloudFront Console

**Access CloudFront**
```bash
AWS Console → Search "CloudFront" → CloudFront Dashboard
```

**Examine Your Distribution**
1. Click **"Distributions"** in the left navigation (if not already there)
2. You should see one existing distribution
3. Click the **Distribution ID link** to open it

### 1.2 Explore Distribution Configuration

**General Tab Deep Dive**
1. Start with the **"General"** tab
2. **Important**: Copy these values to your notepad:
   - **Distribution ARN** (you'll need this later)
   - **Distribution domain name** (looks like `d1234567890abc.cloudfront.net`)

**Test Current Setup**
1. Copy the **Distribution domain name**
2. Open a new browser tab and paste the domain
3. You should see a simple web page served through CloudFront

**🔍 What Just Happened?**
You just accessed content through AWS's global CDN! That web page was served from the nearest edge location, not the original server. Notice how fast it loaded!

**Origins Tab Analysis**
1. Click the **"Origins"** tab
2. You'll see one origin - an Application Load Balancer (ALB)
3. **Copy the Origin domain** value (this is your load balancer's DNS name)

**Test Direct Origin Access**
1. Open a new tab and paste the load balancer DNS
2. You'll see the same content, but served directly from the origin
3. This demonstrates the difference between CDN and direct access

**Behaviors Tab Overview**
1. Click the **"Behaviors"** tab
2. Review the default behavior settings
3. Note how it handles HTTP/HTTPS requests and caching

**🔍 What Just Happened?**
You've seen how CloudFront acts as an intelligent proxy, caching content from origins and serving it globally. The current setup only has one origin (ALB), but we're about to add S3!

---

## Task 2: Create Your S3 Origin Bucket

Time to create a high-performance S3 bucket that will serve as our static content origin. This bucket will store images, CSS, JavaScript, and other static assets.

### 2.1 Create the S3 Bucket

**Navigate to S3**
```bash
AWS Console → Search "S3" → S3 Dashboard
```

**Create Your Bucket**
1. Click **"Create bucket"**
2. **Bucket name**: Use the `LabBucketName` from your lab instructions
3. **Region**: Ensure it matches your `PrimaryRegion`
4. Leave all other settings as default
5. Click **"Create bucket"**

**🔍 What Just Happened?**
You've created a bucket with AWS's default security settings - it's private by default. This is exactly what we want since we'll control access through CloudFront using Origin Access Control (OAC).

---

## Task 3: Configure Public Access (Temporary)

We'll start by making the bucket publicly accessible, then later secure it properly with CloudFront OAC. This approach helps you understand the security journey.

### 3.1 Allow Public Policies

**Modify Block Settings**
1. Click your new bucket name to enter it
2. Navigate to the **"Permissions"** tab
3. Find **"Block public access (bucket settings)"**
4. Click **"Edit"**
5. **Uncheck** "Block all public access"
6. Click **"Save changes"**
7. Type `confirm` when prompted

### 3.2 Create Public Read Policy

**Get Bucket Information**
1. Still in **"Permissions"** tab, find **"Bucket policy"**
2. Click **"Edit"**
3. **Important**: Copy the **Bucket ARN** shown above the policy editor

**Apply Public Policy**
Copy this JSON policy template:

```json
{
    "Version": "2012-10-17",
    "Id": "Policy1621958846486",
    "Statement": [
        {
            "Sid": "OriginalPublicReadPolicy",
            "Effect": "Allow",
            "Principal": "*",
            "Action": [
                "s3:GetObject",
                "s3:GetObjectVersion"
            ],
            "Resource": "YOUR_BUCKET_ARN/*"
        }
    ]
}
```

**Customize and Apply**
1. Replace `YOUR_BUCKET_ARN` with your actual bucket ARN
2. **Critical**: Add `/*` at the end (e.g., `arn:aws:s3:::my-bucket/*`)
3. Paste the complete policy into the editor
4. Click **"Save changes"**

**🔍 What Just Happened?**
You've created a policy that allows anyone on the internet to read objects from your bucket. This is temporary - we'll lock it down with CloudFront OAC later!

---

## Task 4: Upload and Test Content

Let's add some content to test our setup and verify public access works.

### 4.1 Create Organized Structure

**Create Folder**
1. Click the **"Objects"** tab
2. Click **"Create folder"**
3. **Folder name**: `CachedObjects`
4. Click **"Create folder"**

### 4.2 Upload Test Content

**Download Test Image**
1. Right-click this link and save: [AWS Logo](https://d1.awsstatic.com/logos/aws-logo-lockups/poweredbyaws/PB_AWS_logo_RGB_stacked_REV_SQK.91cd4af40773cbfbd29f78e2cc8dfbcc41f90d69.png)
2. Save it as `logo.png` on your computer

**Upload to S3**
1. Click the **"CachedObjects/"** folder you just created
2. Click **"Upload"**
3. Click **"Add files"**
4. Select your `logo.png` file
5. Click **"Upload"**

### 4.3 Test Public Access

**Verify Direct S3 Access**
1. Click the `logo.png` file name
2. In the object details, click the **"Object URL"**
3. A new tab should open showing your image

**🔍 What Just Happened?**
You've successfully uploaded content to S3 and verified it's publicly accessible. Notice the URL structure - it's a direct S3 URL. Soon we'll serve this same content through CloudFront's global network!

---

## Task 5: Secure with CloudFront Origin Access Control

Now for the security magic! We'll configure Origin Access Control (OAC) so only CloudFront can access your S3 bucket, preventing users from bypassing your CDN.

### 5.1 Update Bucket Policy for CloudFront Only

**Create Secure Policy**
1. Return to your bucket's **"Permissions"** tab
2. Click **"Edit"** in the bucket policy section
3. **Replace the entire policy** with this CloudFront-only policy:

```json
{
    "Version": "2012-10-17",
    "Statement": {
        "Sid": "AllowCloudFrontServicePrincipalReadOnly",
        "Effect": "Allow",
        "Principal": {
            "Service": "cloudfront.amazonaws.com"
        },
        "Action": [
            "s3:GetObject",
            "s3:GetObjectVersion"
        ],
        "Resource": "YOUR_BUCKET_ARN/*",
        "Condition": {
            "StringEquals": {
                "AWS:SourceArn": "YOUR_CLOUDFRONT_DISTRIBUTION_ARN"
            }
        }
    }
}
```

**Customize the Policy**
1. Replace `YOUR_BUCKET_ARN` with your bucket ARN + `/*`
2. Replace `YOUR_CLOUDFRONT_DISTRIBUTION_ARN` with your CloudFront distribution ARN
3. Click **"Save changes"**

### 5.2 Block Public Access

**Re-enable Security**
1. Find **"Block public access (bucket settings)"**
2. Click **"Edit"**
3. **Check** "Block all public access"
4. Click **"Save changes"**
5. Type `confirm` when prompted

**🔍 What Just Happened?**
You've locked down your S3 bucket! Now only CloudFront can access it using the specific distribution ARN. Users can no longer bypass CloudFront to access your content directly.

### 5.3 Add S3 as CloudFront Origin

**Navigate to CloudFront**
1. Return to **CloudFront Console**
2. Click your distribution ID
3. Go to the **"Origins"** tab
4. Click **"Create origin"**

**Configure S3 Origin**
```yaml
Origin domain: [Select your bucket from the dropdown]
Origin path: [Leave empty]
Name: My Amazon S3 Origin
Origin access: Origin access control settings (recommended)
```

**Create Origin Access Control**
1. Click **"Create new OAC"**
2. Accept the default settings in the popup
3. Click **"Create"** in the OAC popup
4. Click **"Create origin"** to finish

**🔍 What Just Happened?**
You've added your S3 bucket as a secure origin to CloudFront with OAC protection. CloudFront can now serve content from S3, but direct S3 access is blocked!

---

## Task 6: Configure Cache Behavior for S3 Content

Now we need to tell CloudFront how to handle requests for S3 content. We'll create a behavior that specifically handles PNG files from our CachedObjects folder.

### 6.1 Create S3 Content Behavior

**Add New Behavior**
1. Go to the **"Behaviors"** tab
2. Click **"Create behavior"**

**Configure Behavior Settings**
```yaml
Path pattern: CachedObjects/*.png
Origin: My Amazon S3 Origin
Cache policy: CachingOptimized
Origin request policy: [Leave as default]
Viewer protocol policy: [Leave as default]
```

**Create and Save**
1. Review your settings
2. Click **"Create behavior"**

**🔍 What Just Happened?**
You've created a rule that tells CloudFront: "When someone requests a PNG file from the CachedObjects folder, serve it from the S3 origin with optimized caching." This means faster load times and reduced origin server load!

---

## Task 7: Test Your Secure CDN Setup

Time to verify everything works perfectly! We'll test both scenarios - direct S3 access (should fail) and CloudFront access (should work).

### 7.1 Verify S3 Direct Access is Blocked

**Test Direct S3 URL**
1. Return to **S3 Console**
2. Navigate to your bucket → `CachedObjects/` → `logo.png`
3. Click the **"Object URL"**
4. You should see an **"Access Denied"** error!

**🔍 What Just Happened?**
Perfect! This confirms our security is working. Users can't bypass CloudFront to access content directly from S3.

### 7.2 Test CloudFront Access Works

**Wait for Distribution Update**
1. Return to **CloudFront Console**
2. Check your distribution's **"Status"** - it should be "Enabled"
3. If "Last modified" shows recent activity, wait 2-3 minutes for propagation

**Test CloudFront URL**
1. Copy your **CloudFront distribution domain name**
2. In a new tab, construct this URL:
   ```
   https://your-distribution-domain.cloudfront.net/CachedObjects/logo.png
   ```
3. Your image should load perfectly!

**Performance Test**
1. Refresh the CloudFront URL several times
2. Notice how fast it loads after the first request (caching in action!)
3. Compare to the original load balancer content

**🔍 What Just Happened?**
Excellent! Your secure, high-performance CDN is working perfectly. Content is served from CloudFront's global edge locations while S3 remains secure behind OAC protection.

---

## Task 8: Monitor and Optimize Performance

Let's explore how to monitor your CloudFront distribution and understand its performance characteristics.

### 8.1 Examine CloudFront Metrics

**Access Monitoring**
1. In your CloudFront distribution, click the **"Monitoring"** tab
2. Review available metrics:
   - **Requests**: Number of requests to your distribution
   - **BytesDownloaded**: Data transfer volume
   - **4xxErrorRate**: Client error percentage
   - **5xxErrorRate**: Server error percentage

**Cache Performance**
1. Look for cache hit ratio metrics
2. Higher cache hit ratios = better performance and lower costs
3. Monitor origin load reduction

### 8.2 Understand Cache Behaviors

**Cache Headers Analysis**
1. Use browser developer tools (F12)
2. Load your CloudFront URL
3. Check the **Network** tab
4. Look for cache-related headers:
   - `X-Cache`: Shows HIT, MISS, or RefreshHit
   - `X-Amz-Cf-Pop`: Shows which edge location served the request
   - `Age`: Shows how long the object has been cached

**🔍 What Just Happened?**
You're now monitoring real CDN performance! The `X-Cache: Hit` header means your content was served from cache, providing faster delivery and reducing origin server load.

---

## Advanced Features & Optional Tasks

### Optional Task 1: Cross-Region Replication

Want to add disaster recovery for your S3 content? Let's set up cross-region replication!

**Enable Versioning on Source Bucket**
1. Navigate to your bucket's **"Properties"** tab
2. Find **"Bucket Versioning"**
3. Click **"Edit"** → **"Enable"**
4. Click **"Save changes"**

**Create Destination Bucket**
1. Switch to your **SecondaryRegion** (check lab instructions)
2. Create bucket with name from `DestinationBucketName` in lab instructions
3. Enable versioning on the new bucket
4. Configure public read policy (for testing only)

**Set Up Replication**
1. Return to your original bucket
2. Go to **"Management"** tab
3. Find **"Replication rules"**
4. Click **"Create replication rule"**
5. Configure:
   ```yaml
   Rule name: MyCrossRegionReplication
   Status: Enabled
   Scope: Apply to all objects
   Destination: [Your destination bucket]
   IAM Role: S3CRRRole
   ```

**Test Replication**
1. Upload a new file to your source bucket
2. Check the destination bucket in a few minutes
3. The file should appear automatically!

### Optional Task 2: Add Custom Error Pages

**Create Custom 404 Page**
1. In CloudFront, go to **"Error Pages"** tab
2. Click **"Create Custom Error Response"**
3. Configure:
   ```yaml
   HTTP Error Code: 404
   Error Caching TTL: 300
   Custom Error Response: Yes
   Response Page Path: /error.html
   HTTP Response Code: 404
   ```

---

## Troubleshooting Guide

### Common Issues and Solutions

**Issue: CloudFront not serving S3 content**
```bash
Symptoms: 403 Forbidden or timeout errors
Solutions:
1. Verify bucket policy allows cloudfront.amazonaws.com service
2. Check Distribution ARN matches bucket policy condition
3. Ensure OAC is created and associated with origin
4. Wait 5-15 minutes for global propagation
```

**Issue: Direct S3 access still works**
```bash
Symptoms: Can access S3 URLs directly despite OAC
Solutions:
1. Verify "Block all public access" is enabled
2. Remove any public read policies
3. Ensure only CloudFront service principal has access
4. Check bucket policy syntax is correct
```

**Issue: Cache not working properly**
```bash
Symptoms: Content always loads slowly, no cache hits
Solutions:
1. Check cache behavior path patterns match your content
2. Verify cache policy is set to "CachingOptimized"
3. Use browser dev tools to check X-Cache headers
4. Clear browser cache to test properly
```

**Issue: Behavior not matching content**
```bash
Symptoms: 404 errors for specific file types
Solutions:
1. Check path pattern syntax (e.g., *.png vs CachedObjects/*.png)
2. Verify origin is correctly assigned to behavior
3. Ensure folder structure matches path patterns
4. Test with exact file paths
```

### Debugging Commands

**Check cache status:**
```bash
# Use curl to see headers
curl -I https://your-distribution.cloudfront.net/CachedObjects/logo.png

# Look for these headers:
X-Cache: Hit from cloudfront (cached)
X-Cache: Miss from cloudfront (not cached)
X-Amz-Cf-Pop: Edge location that served the request
```

**Test origin access:**
```bash
# This should fail (403 Forbidden)
curl https://your-bucket.s3.amazonaws.com/CachedObjects/logo.png

# This should work
curl https://your-distribution.cloudfront.net/CachedObjects/logo.png
```

---

## Real-World Applications

### When to Use This Architecture

**E-commerce Platforms**
- **Product Images**: Serve high-resolution product photos globally
- **Static Assets**: CSS, JavaScript, fonts for consistent branding
- **Seasonal Content**: Handle traffic spikes during sales events
- **User Uploads**: Profile pictures, reviews, user-generated content

**Media & Entertainment**
- **Video Streaming**: Distribute video content with adaptive bitrate
- **Image Galleries**: Photo sharing with fast load times worldwide
- **Software Distribution**: Download files, installers, patches
- **Gaming Assets**: Game assets, updates, downloadable content

**Corporate Websites**
- **Marketing Sites**: Fast-loading websites for global audiences
- **Documentation**: Technical docs, PDFs, training materials
- **Brand Assets**: Logos, presentations, marketing collateral
- **Mobile Apps**: API responses, app updates, content delivery

### Performance & Cost Benefits

**Performance Improvements:**
```
Traditional Setup (Direct S3):
├── Global users → Single S3 region
├── Latency: 200-2000ms depending on distance
├── Bandwidth: Limited by single region capacity
└── Reliability: Single point of failure

CloudFront Setup:
├── Global users → Nearest edge location
├── Latency: 20-100ms (90%+ improvement)
├── Bandwidth: Massive global capacity
└── Reliability: 200+ edge locations
```

**Cost Optimization:**
- **Reduced Origin Load**: 80-95% cache hit rates reduce S3 requests
- **Data Transfer Savings**: CloudFront pricing often lower than S3 data transfer
- **Regional Pricing**: Take advantage of edge location regional pricing
- **Compression**: Automatic Gzip compression reduces bandwidth costs

### Security Benefits

**Origin Protection:**
- **OAC Security**: Only CloudFront can access S3 content
- **DDoS Protection**: AWS Shield Standard included
- **SSL/TLS**: Automatic HTTPS with AWS certificates
- **Access Logs**: Detailed logging for security analysis

**Advanced Security Options:**
- **AWS WAF Integration**: Block malicious requests at edge
- **Geographic Restrictions**: Block content by country
- **Signed URLs/Cookies**: Control access to premium content
- **Field-Level Encryption**: Encrypt sensitive form data

---

## Cleanup Instructions

**⚠️ Important**: Follow these steps to avoid ongoing charges!

### Step 1: Disable CloudFront Distribution
```bash
1. CloudFront Console → Distributions
2. Select your distribution
3. General tab → Edit
4. Status: Disabled
5. Save changes
6. Wait for status to become "Disabled" (15-20 minutes)
7. Then delete the distribution
```

### Step 2: Clean Up S3 Resources
```bash
1. S3 Console → Your bucket
2. Objects tab → Select all objects
3. Delete all objects
4. Delete the bucket itself
5. If using cross-region replication, repeat for destination bucket
```

### Step 3: Verify Cleanup
```bash
1. Check CloudFront distributions list is empty
2. Verify S3 buckets are deleted
3. Review billing dashboard to confirm no ongoing charges
```

---

## Congratulations! 🎉

You've successfully built a secure, high-performance global content delivery network! Here's what you accomplished:

### 🌍 **Global CDN Mastery**
- Created a CloudFront distribution with multiple origins
- Implemented Origin Access Control for maximum security
- Configured intelligent cache behaviors for different content types
- Built a system that serves content from 200+ global edge locations

### 🔒 **Security Excellence**
- Secured S3 content behind CloudFront using OAC
- Prevented direct S3 access while maintaining CloudFront functionality
- Implemented proper IAM policies for service-to-service communication
- Created a defense-in-depth security model

### ⚡ **Performance Optimization**
- Achieved 90%+ latency reduction through edge caching
- Implemented content-specific cache behaviors
- Reduced origin server load through intelligent caching
- Built a system that scales automatically with demand

### 🛠️ **Production Skills**
- Multi-origin CloudFront configuration
- Cross-region replication for disaster recovery
- Performance monitoring and optimization techniques
- Real-world security and access control patterns

### What's Next?

Ready to take your CDN skills even further? Consider exploring:

- **Advanced Caching**: Custom cache policies and TTL optimization
- **Lambda@Edge**: Run code at edge locations for dynamic content
- **AWS WAF Integration**: Advanced security and bot protection
- **Real User Monitoring**: Performance analytics and optimization
- **Certificate Management**: Custom SSL certificates and domain names

### Key Takeaways

✅ **Global Performance**: CloudFront dramatically improves user experience worldwide
✅ **Security First**: OAC provides secure content delivery without compromising performance
✅ **Cost Effective**: Reduced origin load and optimized data transfer costs
✅ **Scalable**: Automatic scaling to handle any traffic volume
✅ **Reliable**: Built-in redundancy across 200+ global edge locations

You now have the skills to build enterprise-grade content delivery networks that provide blazing-fast performance, rock-solid security, and global scalability. These are the same patterns used by major websites serving millions of users daily!

**Keep optimizing, keep securing, and remember**: The fastest website is the one that feels instant to every user, everywhere. You've built exactly that! 🚀

---

*© 2025 AWS Solutions Architecture Lab Series. This lab demonstrates production-grade CDN patterns used by global enterprises to deliver content at scale.*