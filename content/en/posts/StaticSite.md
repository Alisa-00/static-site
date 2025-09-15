---
slug: staticsite
date: 2025-09-14T11:46:06+03:00
# lastmod: 2025-09-14T11:46:06+03:00 # Last modification date
tags:
- AWS
- Serverless
- CI/CD
categories:
- Cloud
title:  Automating static site deployment with CI/CD on AWS cloud.
---

I recently put together a simple pipeline for generating and hosting a static website completely on AWS, with minimal . This setup takes advantage of S3 for safe storage and static site hosting, cloudfront for low latency distribution and a Lambda function for automatic update of the static site upon any change on the source files, such as a new post.
This lets me write blog entries in markdown files and simply drop them into the source S3 bucket, this update will kick off the lambda function and it will :

## Creating the source files for the site
I decided to keep it simple and use a template for an already existing static site generator and customize those instead of making my own from scratch. I used [Hugo](https://gohugo.io/) for a static site generator, and use [Yue](https://github.com/CyrusYip/hugo-theme-yue) as the template.
After spending some time customizing everything to my liking, I was ready to upload this to the Input S3 bucket.

### Create the input S3 bucket
Simply upload the source files to a new S3 bucket. This will be the source bucket where we upload new posts or other changes we want to apply to the static site.
We will also need to create an event notification to run the lambda function, and also assign the appropriate permissions to make sure the lambda function can read from the bucket, but we can take care of those later.

### Create the output S3 bucket
Same as the previous bucket. This bucket will be our destination bucket, where the files for the static site itself are stored and served to the public internet. We can deactivate "Block all public access" which will let us access the site through a url. Let's take the chance here to make sure the site is looking just like we expect it to, before making further setups in our cloud.

## Create the CloudFront distribution
Now 

## Automating with AWS Lambda
Lambda is where the automation comes in. I set up an AWS Lambda function to run my Python generator whenever new files are added or existing ones are updated in the bucket.

### Create an execution role for the lambda function
Security best practices: create a new IAM policy for the role for reading from the source bucket and writing to the public bucket.
After this you can create the role, AWS Service: Lambda, and the permission policy you just created.

### Create the lambda layer
Lambda layer contains the hugo binary, this way we can easily execute it without needing to build the code.

Steps:
1. Create a new Lambda function in the AWS Console.  
   - Runtime: **Python 3.x**  
   - Add the Python script as the handler (zip it up with dependencies and upload).  
2. Give the Lambda function an IAM role with permissions to **read from the source S3 bucket** and **write to an output bucket** (where the generated site will live).  
3. In the S3 bucket (`my-static-site-src`), go to **Properties → Event notifications**.  
   - Add an event notification for `s3:ObjectCreated:*` and `s3:ObjectRemoved:*`.  
   - Set the destination as the Lambda function.  
4. Now, whenever I upload or modify Markdown files in the bucket, the Lambda runs, generates static HTML, and writes the new files to the output bucket (`my-static-site-build`).

## Hosting the Static Site
For hosting, there are two main options:

### Option A: S3 Static Website Hosting
1. Enable **Static Website Hosting** on the output bucket (`my-static-site-build`).  
2. Choose “Use this bucket to host a website” and specify an index document (e.g., `index.html`).  
3. Make the bucket contents public (or, better, use CloudFront to handle HTTPS and caching).  
4. Point a custom domain (via Route 53 or another DNS provider) to the bucket or CloudFront distribution.

### Option B: CloudFront Distribution
1. Create a CloudFront distribution with the output bucket (`my-static-site-build`) as the origin.  
2. Attach a custom domain in the distribution settings.  
3. Use **AWS Certificate Manager (ACM)** to issue a free SSL certificate for HTTPS.  
4. Update DNS records (Route 53 or external provider) to point the domain to the CloudFront distribution.

## Final Thoughts
With this setup, all I need to do is write posts in Markdown and drop them into S3. The Lambda function handles generation, and S3 (optionally with CloudFront) serves the site to the world. No manual builds, no servers to manage—just a simple automated publishing flow on AWS.

