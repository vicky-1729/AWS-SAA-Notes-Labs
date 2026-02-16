# Lab 5: Build a Serverless Architecture

*Estimated Time: 75-90 minutes*

## Welcome to the Serverless Revolution! 🚀

Hey there, cloud innovator! Ready to build something that scales infinitely, costs almost nothing when idle, and never requires server maintenance? Welcome to the world of serverless architecture – where your code runs in the cloud without you ever thinking about servers!

Today we're building an intelligent image processing pipeline that responds to events in real-time. Think Instagram meets AWS – every time someone uploads a photo, our serverless army automatically creates thumbnails and mobile-optimized versions. No servers to manage, no capacity planning, just pure, event-driven magic!

## What We're Building Today

```
┌──────────────────── SERVERLESS IMAGE PIPELINE ────────────────────┐
│                                                                     │
│  📸 Upload Image    ┌─────────────┐    📢 S3 Event                │
│  ───────────────►   │  S3 Bucket  │ ──────────────────┐            │
│                     │   (ingest)  │                   │            │
│                     └─────────────┘                   ▼            │
│                                               ┌──────────────┐      │
│                                               │ SNS Topic    │      │
│                                               │ (Broadcast)  │      │
│                                               └──────┬───────┘      │
│                                                      │              │
│                              ┌───────────────────────┼──────────────┐
│                              │                       │              │
│                              ▼                       ▼              │
│                     ┌─────────────────┐    ┌─────────────────┐      │
│                     │ SQS Queue       │    │ SQS Queue       │      │
│                     │ (thumbnail)     │    │ (mobile)        │      │
│                     └─────────┬───────┘    └─────────┬───────┘      │
│                               │                      │              │
│                               ▼                      ▼              │
│                     ┌─────────────────┐    ┌─────────────────┐      │
│                     │ Lambda Function │    │ Lambda Function │      │
│                     │ (CreateThumb)   │    │ (CreateMobile)  │      │
│                     └─────────┬───────┘    └─────────┬───────┘      │
│                               │                      │              │
│                               └──────┬───────────────┘              │
│                                      ▼                              │
│                              ┌─────────────────┐                    │
│                              │  S3 Bucket      │                    │
│                              │ (processed)     │                    │
│                              │ /thumbnail/     │                    │
│                              │ /mobile/        │                    │
│                              └─────────────────┘                    │
│                                                                     │
│  📊 Monitor Everything with CloudWatch Logs & Metrics              │
└─────────────────────────────────────────────────────────────────────┘
```

This isn't just image processing – it's a masterclass in event-driven, decoupled architecture that can handle millions of images with zero server management!

## Learning Objectives

By completing this lab, you'll master:

- **Event-Driven Architecture**: Build systems that react automatically to events
- **Serverless Computing**: Create Lambda functions that scale automatically
- **Message Queuing**: Use SQS for reliable, decoupled communication
- **Pub/Sub Messaging**: Leverage SNS for broadcasting events to multiple subscribers
- **S3 Event Notifications**: Trigger workflows when objects are uploaded
- **Monitoring & Debugging**: Track serverless applications with CloudWatch

## Prerequisites and Setup

### What You'll Need
- AWS Console access with appropriate permissions
- Basic understanding of Python (don't worry, code is provided!)
- A test image file (JPG format)
- Enthusiasm for building scalable, cost-effective solutions!

### Important Notes
💰 **Cost Awareness**: Serverless = pay only for what you use!
🎯 **Region Consistency**: Stay in your assigned region throughout the lab
⚡ **Event-Driven Thinking**: Everything happens in response to events

---

## Task 1: Create Your SNS Broadcasting Hub

Let's start by building our message broadcasting system. Think of SNS as a smart loudspeaker that announces events to all interested listeners.

### 1.1 Build the SNS Topic

**Navigate to Simple Notification Service**

```bash
AWS Console → Search "Simple Notification Service" → SNS Dashboard
```

**Create Your Event Hub**
1. Click **"Topics"** in the left navigation
2. Click **"Create topic"**

**Configure the Topic**
```yaml
Type: Standard
Name: resize-image-topic-[YOUR-4-DIGIT-NUMBER]
```

💡 **Pro Tip**: Add 4 random numbers to your topic name to ensure uniqueness (e.g., `resize-image-topic-1234`)

**Create and Capture Details**
1. Click **"Create topic"**
2. **Important**: Copy these values to a notepad:
   - **Topic ARN**: `arn:aws:sns:region:account:resize-image-topic-XXXX`
   - **Topic Owner**: Your 12-digit AWS Account ID

**🔍 What Just Happened?**
You've created a central hub that can broadcast messages to multiple subscribers simultaneously. When an image is uploaded, this topic will notify all our processing queues at once.

---

## Task 2: Create Your Message Queues

Now let's build the queues that will hold our processing jobs. Each queue serves a different image processing purpose.

### 2.1 Create the Thumbnail Processing Queue

**Navigate to Simple Queue Service**
```bash
AWS Console → Search "Simple Queue Service" → SQS Dashboard
```

**Create the Queue**
1. Click **"Create queue"**
2. Configure:
   ```yaml
   Type: Standard
   Name: thumbnail-queue
   ```
3. Leave all other settings as default
4. Click **"Create queue"**

**Subscribe to SNS Topic**
1. In your new queue's details, click the **"SNS subscriptions"** tab
2. Click **"Subscribe to Amazon SNS topic"**
3. Select your `resize-image-topic-XXXX` from the dropdown
4. Click **"Save"**

### 2.2 Create the Mobile Processing Queue

**Create Second Queue**
1. Navigate back to **"Queues"** in SQS
2. Click **"Create queue"**
3. Configure:
   ```yaml
   Type: Standard
   Name: mobile-queue
   ```
4. Click **"Create queue"**

**Subscribe to SNS Topic**
1. Click the **"SNS subscriptions"** tab
2. Click **"Subscribe to Amazon SNS topic"**
3. Select your `resize-image-topic-XXXX`
4. Click **"Save"**

### 2.3 Test Your Message Flow

Let's verify our pub/sub system works!

**Send a Test Message**
1. Return to **SNS Console → Topics**
2. Click your `resize-image-topic-XXXX`
3. Click **"Publish message"**

**Configure Test Message**
```yaml
Subject: Hello World Test
Message body: Testing our serverless pipeline!
Message attributes:
  - Type: String
  - Name: TestMessage
  - Value: Pipeline Active
```

**Verify Message Delivery**
1. Click **"Publish message"**
2. Navigate to **SQS Console**
3. Select either queue and click **"Send and receive messages"**
4. Click **"Poll for messages"**
5. You should see your test message!

**🔍 What Just Happened?**
Your pub/sub system is working! One message to SNS was delivered to both SQS queues. This fanout pattern allows multiple services to process the same event independently.

---

## Task 3: Configure S3 Event Notifications

Time to make S3 smart! We'll configure it to automatically broadcast events when images are uploaded.

### 3.1 Configure SNS Access Policy

First, we need to give S3 permission to publish to our SNS topic.

**Update Topic Policy**
1. Navigate to **SNS Console → Topics**
2. Select your `resize-image-topic-XXXX`
3. Click **"Edit"**
4. Expand **"Access policy - optional"**

**Replace the Policy**
Delete existing content and paste this policy:

```json
{
  "Version": "2008-10-17",
  "Id": "__default_policy_ID",
  "Statement": [
    {
      "Sid": "__default_statement_ID",
      "Effect": "Allow",
      "Principal": {
        "AWS": "*"
      },
      "Action": [
        "SNS:GetTopicAttributes",
        "SNS:SetTopicAttributes",
        "SNS:AddPermission",
        "SNS:RemovePermission",
        "SNS:DeleteTopic",
        "SNS:Subscribe",
        "SNS:ListSubscriptionsByTopic",
        "SNS:Publish"
      ],
      "Resource": "YOUR_SNS_TOPIC_ARN",
      "Condition": {
        "StringEquals": {
          "AWS:SourceAccount": "YOUR_ACCOUNT_ID"
        }
      }
    },
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "s3.amazonaws.com"
      },
      "Action": "SNS:Publish",
      "Resource": "YOUR_SNS_TOPIC_ARN",
      "Condition": {
        "StringEquals": {
          "AWS:SourceAccount": "YOUR_ACCOUNT_ID"
        }
      }
    }
  ]
}
```

**⚠️ Critical**: Replace both occurrences of:
- `YOUR_SNS_TOPIC_ARN` with your actual topic ARN
- `YOUR_ACCOUNT_ID` with your 12-digit AWS account ID

### 3.2 Configure S3 Event Notifications

**Find Your Lab Bucket**
1. Navigate to **S3 Console**
2. Look for a bucket named like `xxxxx-labbucket-xxxxx`
3. Click on the bucket name

**Create Event Notification**
1. Click the **"Properties"** tab
2. Scroll to **"Event notifications"**
3. Click **"Create event notification"**

**Configure the Event**
```yaml
Event name: resize-image-event
Prefix: ingest/
Suffix: .jpg
Event types: ✅ All object create events
Destination: SNS topic
SNS topic: resize-image-topic-XXXX
```

**🔍 What Just Happened?**
Now S3 will automatically notify our SNS topic whenever a .jpg file is uploaded to the `ingest/` folder. The prefix and suffix filters ensure we only process the right files in the right location.

---

## Task 4: Create Your Lambda Functions

Time for the real magic! Let's create serverless functions that automatically process images.

### 4.1 Create the Thumbnail Generator

**Navigate to Lambda Console**
```bash
AWS Console → Search "Lambda" → Lambda Dashboard
```

**Create Function**
1. Click **"Create function"**
2. Select **"Author from scratch"**

**Configure Function Basics**
```yaml
Function name: CreateThumbnail
Runtime: Python 3.9
Architecture: x86_64
```

**Set Execution Role**
1. Expand **"Change default execution role"**
2. Select **"Use an existing role"**
3. Choose role named like `XXXXX-LabExecutionRole-XXXXX`

**⚠️ Critical**: Must use Python 3.9 – the code is specifically written for this version!

### 4.2 Configure Thumbnail Function Trigger

**Add SQS Trigger**
1. Click **"Add trigger"**
2. Configure:
   ```yaml
   Source: SQS
   SQS Queue: thumbnail-queue
   Batch size: 1
   ```
3. Click **"Add"**

**Deploy the Code**
1. Click the **"Code"** tab
2. Click **"Upload from" → "Amazon S3 location"**
3. Use the `CreateThumbnailZIPLocation` URL from your lab instructions
4. Click **"Save"**

**Configure Runtime Settings**
1. Scroll to **"Runtime settings"**
2. Click **"Edit"**
3. Set **Handler**: `CreateThumbnail.handler`
4. Click **"Save"**

**Add Function Description**
1. Click **"Configuration"** tab
2. Click **"General configuration"**
3. Click **"Edit"**
4. Description: `Create a thumbnail-sized image (128x128)`
5. Click **"Save"**

### 4.3 Create the Mobile Image Generator

**Create Second Function**
1. Return to Lambda console
2. Click **"Create function"**
3. Select **"Author from scratch"**

**Configure Function Basics**
```yaml
Function name: CreateMobileImage
Runtime: Python 3.9
Execution role: Use existing role → XXXXX-LabExecutionRole-XXXXX
```

**Add SQS Trigger**
1. Click **"Add trigger"**
2. Configure:
   ```yaml
   Source: SQS
   SQS Queue: mobile-queue
   Batch size: 1
   ```

**Deploy the Code**
1. Click **"Code"** tab
2. Click **"Upload from" → "Amazon S3 location"**
3. Use the `CreateMobileImageZIPLocation` URL from your lab instructions
4. Click **"Save"**

**Configure Runtime Settings**
1. Set **Handler**: `CreateMobileImage.handler`
2. Description: `Create a mobile-friendly image (640x320)`

### 4.4 Understanding the Lambda Code

Here's what your functions do (code is pre-packaged):

**CreateThumbnail Function Logic:**
```python
# Key operations:
1. Receive SQS message with S3 event details
2. Download original image from S3
3. Resize to 128x128 pixels using PIL
4. Upload to S3 in /thumbnail/ folder
5. Clean up temporary files
```

**CreateMobileImage Function Logic:**
```python
# Key operations:
1. Receive SQS message with S3 event details
2. Download original image from S3
3. Resize to 640x320 pixels using PIL
4. Upload to S3 in /mobile/ folder
5. Clean up temporary files
```

**🔍 What Just Happened?**
You've created two serverless functions that automatically process images when messages appear in their respective queues. No servers to manage, automatic scaling, and you only pay when they run!

---

## Task 5: Test Your Serverless Pipeline

Time for the moment of truth! Let's upload an image and watch the magic happen.

### 5.1 Prepare Your Test Image

**Download a Test Image**
Choose one of these options:
- Right-click [AWS Logo](https://docs.aws.amazon.com/images/lambda/latest/dg/images/sample-app-logo.png) → Save as `TestImage.jpg`
- Use any JPG image you have
- Download from: [Sample Images](https://picsum.photos/800/600.jpg)

**⚠️ Important**: 
- Save as `.jpg` extension (not .jpeg)
- Keep file size reasonable (under 5MB)
- Use a descriptive name like `TestImage.jpg`

### 5.2 Upload and Trigger the Pipeline

**Navigate to S3**
1. Go to **S3 Console**
2. Click your `xxxxx-labbucket-xxxxx` bucket
3. Click the **`ingest/`** folder (create if it doesn't exist)

**Upload Your Image**
1. Click **"Upload"**
2. Click **"Add files"**
3. Select your test image
4. Click **"Upload"**

**Watch the Magic Begin!**
The moment you upload, this happens:
1. S3 detects the upload event
2. S3 publishes notification to SNS
3. SNS broadcasts to both SQS queues
4. Lambda functions automatically trigger
5. Images are processed and stored

### 5.3 Monitor Real-Time Processing

**Check Lambda Execution**
1. Navigate to **Lambda Console**
2. Click **"CreateThumbnail"**
3. Click **"Monitor"** tab
4. You should see invocation metrics!

**View Detailed Logs**
1. Click **"View CloudWatch logs"**
2. Click the most recent log stream
3. Expand log entries to see processing details

**Example Log Output:**
```
START RequestId: abc123...
your-bucket-name
ingest/TestImage.jpg
Image downloaded successfully
Thumbnail created: 128x128
Upload completed to thumbnail/Thumbnail-TestImage.jpg
END RequestId: abc123...
REPORT Duration: 2345.67 ms Billed Duration: 2400 ms Memory Size: 128 MB Max Memory Used: 89 MB
```

---

## Task 6: Validate Your Results

Let's confirm everything worked perfectly!

### 6.1 Check Processed Images

**Verify Thumbnail Creation**
1. Return to **S3 Console**
2. Navigate to your bucket
3. Look for a **`thumbnail/`** folder
4. You should see `Thumbnail-TestImage.jpg`
5. Click to preview – it should be 128x128 pixels

**Verify Mobile Image Creation**
1. Look for a **`mobile/`** folder
2. You should see `MobileImage-TestImage.jpg`
3. Click to preview – it should be 640x320 pixels

### 6.2 Analyze CloudWatch Metrics

**Lambda Performance Metrics**
1. Navigate to **Lambda Console**
2. Select either function
3. Check **"Monitor"** tab for:
   - **Invocations**: Should show 1 (or more if you tested multiple times)
   - **Duration**: Typical 2-5 seconds for image processing
   - **Error count**: Should be 0
   - **Success rate**: Should be 100%

**SQS Queue Metrics**
1. Navigate to **SQS Console**
2. Select either queue
3. Check **"Monitoring"** tab for:
   - **Messages Sent**: Should show received messages
   - **Messages Received**: Should show Lambda consumption

### 6.3 Test Different Scenarios

**Try Different Image Formats**
1. Upload a .png file – should be ignored (suffix filter)
2. Upload to root bucket – should be ignored (prefix filter)
3. Upload multiple JPGs to `ingest/` – all should process

**🔍 What Just Happened?**
You've built a production-ready, event-driven image processing pipeline! It automatically scales, processes images in parallel, and costs almost nothing when not in use.

---

## Advanced Features & Optional Tasks

### Optional Task 1: Add S3 Lifecycle Management

Let's automatically clean up old files to save costs.

**Configure Lifecycle Rule**
1. Navigate to **S3 Console → Your bucket**
2. Click **"Management"** tab
3. Click **"Create lifecycle rule"**

```yaml
Rule name: cleanup-old-files
Rule scope: Limit scope using filters
Prefix: ingest/
Actions: ✅ Expire current versions of objects
Days after creation: 30
```

This automatically deletes files from the ingest folder after 30 days.

### Optional Task 2: Add Email Notifications

Get notified when images are processed!

**Add Email Subscription**
1. Navigate to **SNS Console → Subscriptions**
2. Click **"Create subscription"**
3. Configure:
   ```yaml
   Topic ARN: [Your topic ARN]
   Protocol: Email
   Endpoint: your-email@example.com
   ```
4. Check your email and confirm the subscription

Now you'll get an email every time an image is uploaded!

### Optional Task 3: Add Error Handling

**Create Dead Letter Queue**
1. Create a new SQS queue: `failed-processing-queue`
2. In your Lambda functions:
   - Configuration → Asynchronous invocation
   - Set dead letter queue for failed processing

---

## Troubleshooting Guide

### Common Issues and Solutions

**Issue: Lambda function not triggering**
```bash
Symptoms: Images upload but no processing happens
Solutions:
1. Check SQS trigger is enabled in Lambda console
2. Verify IAM role has SQS permissions
3. Check S3 event notification configuration
4. Confirm SNS topic policy allows S3 to publish
```

**Issue: Lambda function errors**
```bash
Symptoms: Function executes but fails
Solutions:
1. Check CloudWatch logs for specific error messages
2. Verify handler name matches function file structure
3. Confirm Python runtime version is 3.9
4. Check IAM role has S3 read/write permissions
```

**Issue: Images not appearing in processed folders**
```bash
Symptoms: Function runs successfully but no output
Solutions:
1. Check S3 bucket permissions
2. Verify function code is uploading to correct path
3. Look for case sensitivity in folder names
4. Check if images are in different AWS region
```

**Issue: SNS messages not reaching SQS**
```bash
Symptoms: S3 events trigger SNS but queues stay empty
Solutions:
1. Verify SQS subscriptions are confirmed
2. Check SNS access policy allows S3 publishing
3. Confirm topic ARN is correct in S3 event notification
4. Test SNS publishing manually first
```

### Debugging Commands

**Check Lambda logs:**
```bash
# Navigate to CloudWatch → Log groups → /aws/lambda/CreateThumbnail
# Look for recent log streams and error messages
```

**Test SQS manually:**
```bash
# In SQS console, send a test message to verify queue is working
# Check "Dead letter queue" for failed messages
```

**Verify S3 event configuration:**
```bash
# S3 → Bucket → Properties → Event notifications
# Ensure prefix="ingest/" and suffix=".jpg" are correct
```

---

## Real-World Applications

### When to Use This Architecture

**Media Processing Platforms**
- **Social Media**: Automatically resize profile pictures and posts
- **E-commerce**: Generate product thumbnails and zoom images
- **News Sites**: Process uploaded photos for web and mobile

**Document Processing**
- **PDF Generation**: Convert uploaded docs to different formats
- **OCR Processing**: Extract text from uploaded images
- **Compliance**: Scan uploads for sensitive information

**Data Pipeline Triggers**
- **ETL Jobs**: Process uploaded CSV files
- **Backup Systems**: Replicate files to multiple regions
- **Analytics**: Trigger data processing when new datasets arrive

### Cost Optimization Benefits

**Traditional vs. Serverless Costs:**

**Traditional Architecture:**
```
EC2 instances running 24/7: $50-200/month
Load balancer: $20/month
Auto Scaling overhead: $10-50/month
Total: $80-270/month (regardless of usage)
```

**Serverless Architecture:**
```
Lambda execution: $0.000016 per 100ms
SQS requests: $0.40 per million requests
SNS notifications: $0.50 per million notifications
S3 storage: $0.023 per GB/month
Total: ~$1-10/month for moderate usage
```

**💰 Cost Benefits**:
- **No idle costs**: Pay only when processing images
- **Automatic scaling**: Handle spikes without pre-provisioning
- **No maintenance**: No servers to patch or monitor

### Performance Characteristics

**Scalability:**
- **Concurrent executions**: Up to 1,000 Lambda functions by default
- **Processing speed**: Typically 2-5 seconds per image
- **Throughput**: Can process hundreds of images per minute

**Reliability:**
- **Built-in retry**: Failed functions automatically retry
- **Dead letter queues**: Capture and analyze failures
- **Multi-AZ**: Automatically distributed across availability zones

---

## Cleanup Instructions

**⚠️ Important**: Clean up to avoid ongoing charges!

### Step 1: Empty S3 Buckets
```bash
1. S3 Console → Your bucket
2. Select all objects in all folders
3. Actions → Delete
4. Confirm deletion
```

### Step 2: Delete Lambda Functions
```bash
1. Lambda Console → Functions
2. Select CreateThumbnail → Actions → Delete
3. Select CreateMobileImage → Actions → Delete
```

### Step 3: Clean Up SQS Queues
```bash
1. SQS Console → Queues
2. Select thumbnail-queue → Delete
3. Select mobile-queue → Delete
```

### Step 4: Remove SNS Resources
```bash
1. SNS Console → Topics
2. Select resize-image-topic-XXXX → Delete
3. SNS Console → Subscriptions
4. Delete any email subscriptions created
```

### Step 5: Remove S3 Event Notifications
```bash
1. S3 Console → Your bucket → Properties
2. Event notifications → Select resize-image-event → Delete
```

### Step 6: Verify Cleanup
```bash
1. Check Lambda functions list is empty
2. Verify SQS queues are deleted
3. Confirm SNS topics are removed
4. Check S3 bucket has no event notifications
```

---

## Congratulations! 🎉

You've successfully built a production-ready serverless architecture! Here's what you accomplished:

### 🏗️ **Serverless Mastery**
- Created event-driven Lambda functions that scale automatically
- Built a pub/sub messaging system with SNS and SQS
- Configured S3 to trigger workflows on file uploads
- Implemented parallel processing with independent queues

### 🔧 **Architecture Excellence**
- Designed a decoupled, resilient system
- Implemented automatic retry and error handling
- Built cost-effective, pay-per-use infrastructure
- Created a system that scales from zero to millions

### 🚀 **Production Skills**
- Monitored serverless applications with CloudWatch
- Implemented proper IAM security policies
- Built event filters for targeted processing
- Created a maintenance-free, self-healing system

### What's Next?

Ready to expand your serverless expertise? Consider exploring:

- **Advanced Lambda**: Custom layers, provisioned concurrency, and performance optimization
- **Step Functions**: Orchestrate complex workflows across multiple services
- **API Gateway**: Add REST APIs to trigger your functions
- **EventBridge**: Advanced event routing and filtering
- **DynamoDB**: Add serverless databases to your architecture

### Key Takeaways

✅ **Serverless = Scalable**: Automatically handles any load without pre-planning
✅ **Event-Driven = Efficient**: Resources activate only when needed
✅ **Decoupled = Resilient**: Components fail independently without affecting the whole system
✅ **Managed Services = Focus**: Spend time on business logic, not infrastructure

You now have the skills to build modern, cloud-native applications that are cost-effective, scalable, and maintainable. These patterns are used by companies processing millions of events daily!

**Keep innovating, keep building, and remember**: The best architectures are the ones you don't have to think about once they're running. You've built exactly that! 💪

---

*© 2025 AWS Solutions Architecture Lab Series. This lab demonstrates industry-standard serverless patterns used in production environments worldwide.*