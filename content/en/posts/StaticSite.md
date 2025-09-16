---
slug: staticsite
date: 2025-09-14T11:46:06+03:00
# lastmod: 2025-09-14T11:46:06+03:00 # Last modification date
tags:
- Cloud
- Serverless
- CI/CD
title:  Automating static site deployment with CI/CD on AWS
summary: A funny thing I tried
---

I recently put together a simple pipeline for generating and hosting a static website completely on AWS, with minimal maintenance work after setup.
This setup takes advantage of S3 for safe storage, cloudfront for hosting with https and a Lambda function for automatic generation and deployment of the site when any change is applied, such as a new post entry being added.
This is convenient because if I wrote a new entry I can simply upload it to the S3 bucket and then the site will be automatically updated. It is a serverless architecture that does not require my attention at all.

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

Still, the question remains. Why create this pipeline, and why on AWS if there are easier and better alternatives for this use case, such as github pages which would be completely free as opposed to potentially having a cost.
And its simply because I wanted to. I felt like doing something on AWS. I was doing some work already to set up this site, and I thought it could be fun to try something like this, its a simple but useful exercise.

While I don't actually use this pipeline to maintain and deploy my site because of its downsides, it is still useful architecture that could be useful for many use cases such as, one example that comes to mind is storing music files unmodified and also a normalized version of the file, where one bucket would store either original or normalized, and the lambda function would apply normalization to the file.
