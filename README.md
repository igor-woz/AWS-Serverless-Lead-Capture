# Serverless Lead Capture on AWS

A fully serverless web application that hosts a static website and captures lead/contact form submissions — sending email notifications via SES and storing data in DynamoDB. This project is a part of Udemy Course: "AWS Cloud Projects for Engineers: 5 Real-World Projects" by Pravin Mishra and is published here to demonstrate its completion and understanding.

## What I learned

By completing this project, I gained hands-on experience with:

**Cloud & Serverless Fundamentals**
- How to design and build a serverless architecture on AWS without managing any servers
- How AWS services integrate with each other to form a complete, production-style application

**Static Website Hosting**
- Hosting a static website on **Amazon S3** with public access policies
- Accelerating global delivery using **Amazon CloudFront** as a CDN with HTTPS (SSL/TLS via ACM)
- Mapping a custom domain using **Amazon Route 53**

**Serverless Backend Development**
- Writing and deploying **AWS Lambda** functions in Node.js using the AWS SDK v3
- Building and configuring a **REST API with API Gateway**, including resource/method setup, CORS configuration, and stage deployment
- Sending transactional emails programmatically using **Amazon SES**

**Data Persistence**
- Creating and querying a **DynamoDB** table (NoSQL) via the AWS CLI and console
- Using the `DynamoDBDocumentClient` with `PutCommand` to store structured form data
- Generating unique record IDs with `randomUUID` and storing ISO timestamps

**Security & IAM Best Practices**
- Creating least-privilege **IAM policies** scoped to specific AWS actions (e.g., `ses:SendEmail`, `dynamodb:PutItem`)
- Attaching **IAM roles** to Lambda functions rather than hardcoding credentials
- Understanding multi-layer access control (bucket policies, IAM, CORS)

**Debugging & Observability**
- Testing API endpoints with **cURL** (both OPTIONS preflight and POST requests)
- Monitoring Lambda execution logs in **Amazon CloudWatch**
- Verifying end-to-end data flow from a browser form submission to DynamoDB

**Frontend Integration**
- Connecting a static HTML form to a serverless backend using the browser `fetch` API
- Handling async form submission with error/success feedback in vanilla JavaScript


## Architecture

```
User → CloudFront → S3 (Static Site)
         ↓
    Contact Form → API Gateway → Lambda → SES (Email)
                                       → DynamoDB (Storage)
```

## AWS Services Used

| Service | Purpose |
|---|---|
| **S3** | Hosts the static website files |
| **CloudFront** | CDN for global distribution with HTTPS |
| **Route 53** | Custom domain DNS management |
| **API Gateway** | REST API endpoint for the contact form (POST) |
| **AWS Lambda** | Processes form submissions (Node.js) |
| **Amazon SES** | Sends email notifications on new submissions |
| **DynamoDB** | Persists contact form entries |
| **IAM** | Roles and policies for Lambda permissions |
| **CloudWatch** | Logging and monitoring for Lambda |

## Prerequisites

- AWS CLI installed and configured
- An AWS account with sufficient permissions
- A verified domain/email in Amazon SES
- Node.js (for Lambda function development)

## Project Structure

```
Ebook/
├── index.html          # Main landing page with contact form
├── 404.html            # Custom error page
├── assets/             # CSS, images, JS
└── ...
```

## Setup & Deployment

### Phase 1 — Host Static Website

1. **Clone the repository**
   ```bash
   git clone https://github.com/pravinmishraaws/Ebook.git
   cd Ebook
   ```

2. **Create an S3 bucket**
   ```bash
   aws s3 mb s3://epicreads
   ```

3. **Enable static website hosting** on the bucket (via AWS Console).

4. **Add a public bucket policy**
   ```json
   {
     "Version": "2012-10-17",
     "Statement": [{
       "Sid": "PublicReadGetObject",
       "Effect": "Allow",
       "Principal": "*",
       "Action": ["s3:GetObject"],
       "Resource": ["arn:aws:s3:::epicreads/*"]
     }]
   }
   ```

5. **Sync website files to S3**
   ```bash
   aws s3 sync . s3://epicreads
   ```

6. **Configure CloudFront** for global CDN delivery with an ACM SSL certificate (us-east-1).

7. **Configure Route 53** to point your custom domain to the CloudFront distribution.

---

### Phase 2 — Dynamic Contact Form

1. **Verify sender/receiver emails** in Amazon SES.

2. **Create IAM Policy** (`EpicReads_SES_policy`) with permissions for SES send and CloudWatch logs, then attach to a new **IAM Role** (`Epicreads_role`).

3. **Create Lambda function** (`epicreads_contactus`) using the provided Node.js code and attach the IAM role.

4. **Create API Gateway** REST API (`EpicReads_api`):
   - Resource: `epicreads_resource`
   - Method: `POST`
   - Enable CORS
   - Deploy to a `dev` stage

5. **Update `index.html`** with the API Gateway endpoint URL in the form's JavaScript fetch call.

6. **Re-sync to S3**
   ```bash
   aws s3 sync . s3://epicreads
   ```

7. **Test the API**
   ```bash
   curl -i -X POST "https://<api-id>.execute-api.us-east-1.amazonaws.com/dev/epicreads_resource" \
     -H "Content-Type: application/json" \
     -d '{"name": "Jane Doe", "phone": "+1-555-1234", "email": "jane@example.com", "message": "Hello!"}'
   ```

---

### Phase 3 — Data Persistence with DynamoDB

1. **Create DynamoDB table**
   ```bash
   aws dynamodb create-table \
     --table-name ContactMessages \
     --attribute-definitions AttributeName=id,AttributeType=S \
     --key-schema AttributeName=id,KeyType=HASH \
     --billing-mode PAY_PER_REQUEST \
     --region us-east-1
   ```

2. **Add inline IAM policy** (`LambdaContact-DdbWrite`) to the Lambda role:
   ```json
   {
     "Version": "2012-10-17",
     "Statement": [{"Effect": "Allow", "Action": ["dynamodb:PutItem"], "Resource": "*"}]
   }
   ```

3. **Update the Lambda function** to write form submissions to DynamoDB (`ContactMessages` table) in addition to sending the SES email.

4. **Deploy and verify** entries appear in DynamoDB after a form submission.

---

##License

This project is for educational purposes. See the original walkthrough for full details.
