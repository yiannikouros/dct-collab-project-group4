# **README.md**

# **DCT Collab Project – Group 4**

Serverless Image Labeling and Retrieval Application

This repository contains the documentation and artifacts for a serverless image‑processing application built using AWS services. The system allows authenticated users to upload images, automatically analyzes them using Amazon Rekognition, stores metadata in Amazon RDS, and displays previously uploaded images along with their labels.

This repo includes:

- Architecture diagram
- SQL schema
- CloudFormation template
- Screenshots or demo video
- Final presentation slides
- Project documentation (this README)

---

## **1. Overview**

This project implements a fully serverless, multi‑region image labeling application. Users authenticate through Amazon Cognito, upload images through a pre‑signed URL, and view previously uploaded images and labels. Image metadata and labels are stored in Amazon RDS with cross‑AZ replication for high availability.

The frontend is hosted on Amazon S3 as a static website. The backend is powered by AWS Lambda, API Gateway, Amazon Rekognition, and Amazon RDS.

---

## **2. Architecture Summary**

The system consists of the following major components:

- **Amazon Route 53**  
  Provides DNS routing and health checks for multi‑region failover.

- **Amazon S3 (Static Website Hosting)**  
  Hosts the frontend HTML pages (Login/Signup, Upload, Gallery).

- **Amazon Cognito**  
  Handles user authentication via AWS‑hosted UI.  
  Returns JWT tokens used for API authorization.

- **Amazon API Gateway**  
  Exposes backend endpoints for:
  - Generating pre‑signed upload URLs
  - Triggering image analysis
  - Retrieving previously uploaded images

- **AWS Lambda Functions**
  - Validate user identity
  - Generate pre‑signed S3 URLs
  - Trigger Rekognition
  - Store metadata in RDS
  - Retrieve image metadata for the gallery

- **Amazon S3 (Image Bucket)**  
  Stores uploaded images.  
  Object creation triggers the analysis Lambda.

- **Amazon Rekognition**  
  Analyzes uploaded images and returns labels + confidence scores.

- **Amazon RDS (with Read Replica)**  
  Stores image metadata, labels, confidence scores, and user identity.  
  Read replica supports high availability and disaster recovery.

---

## **3. User Journeys**

### **User Journey 1 — Login / Signup**

1. User visits the S3‑hosted static website.
2. Route 53 resolves the domain and directs traffic to the correct region.
3. User clicks **Login/Signup**.
4. Cognito displays the AWS‑hosted authentication form.
5. On success, Cognito returns credentials and redirects the user to the home page.
6. The browser now holds a valid JWT token for authenticated API calls.

---

### **User Journey 2 — Upload Image**

1. User clicks **Upload Image** on the S3 website.
2. Browser sends a POST request to API Gateway with the user’s JWT.
3. API Gateway verifies the token with Cognito.
4. Lambda generates a pre‑signed S3 upload URL and returns it.
5. Browser uploads the image directly to the S3 image bucket using the pre‑signed URL.
6. S3 object creation triggers a Lambda function.
7. Lambda calls Amazon Rekognition to analyze the image.
8. Rekognition returns labels and confidence scores.
9. Lambda stores:
   - S3 object key
   - User identity
   - Labels
   - Confidence scores
   - Timestamp  
     in Amazon RDS.

---

### **User Journey 3 — View Previously Uploaded Images**

1. User clicks **See Previous Images**.
2. Browser sends a GET request to API Gateway with the user’s JWT.
3. API Gateway verifies the token with Cognito.
4. Lambda queries RDS for all images belonging to the authenticated user.
5. Lambda generates pre‑signed GET URLs for each image.
6. Lambda returns a JSON response containing:
   - Image URL
   - Labels
   - Confidence scores
   - Upload timestamp
7. Browser renders the gallery using the returned data.

---

## **4. High Availability and Failover**

- Route 53 performs health checks on the primary region.
- If the primary region becomes unavailable, Route 53 redirects traffic to the secondary region.
- RDS uses asynchronous replication to maintain a read replica in another Availability Zone.
- Static website hosting and API endpoints remain accessible during failover.

---

## **5. Database Schema**

The SQL schema used for storing image metadata is included in:

```
/database/schema.sql
```

It defines fields such as:

- user_sub
- object_key
- bucket
- labels (JSON)
- confidence scores
- created_at timestamp

---

## **6. CloudFormation Template**

The CloudFormation template is located in:

```
/cloudformation/template.yaml
```

It defines the infrastructure components required for the application, including:

- S3 buckets
- IAM roles
- Lambda functions
- API Gateway configuration
- RDS instance (optional depending on deployment)

---

## **7. Screenshots / Demo**

Screenshots demonstrating the application flow are located in:

```
/screenshots/
```

These include:

- Login/Signup
- Upload page
- Gallery view
- RDS table entries
- Rekognition output

A video demo may also be included.

---

## **8. Presentation**

The final presentation slides are located in:

```
/presentation/final-presentation.pdf
```

---

## **9. Team Contributions**

- **Backend Development:** Lambda functions, API Gateway, Rekognition integration
- **Database & Schema:** RDS setup, MySQL/PostgreSQL schema
- **Frontend Pages:** Static HTML pages hosted on S3
- **Architecture & Documentation:** Architecture diagram, README, CloudFormation, repo structure
- **Testing & Demo:** Screenshots, video walkthrough, presentation slides
