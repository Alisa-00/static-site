---
slug: foundry
date: 2025-09-14T11:45:39+03:00
# lastmod: 2025-09-14T11:45:39+03:00 # Last modification date
tags:
- AWS
categories:
- Cloud
title: Hosting a Foundry VTT Server on AWS with S3 and CloudFront
---

As another exercise in AWS deployment, I set up a Foundry Virtual Tabletop (VTT) server using EC2, with static file assets offloaded to S3 and delivered through CloudFront for performance and scalability. Here’s how I went about it:

## Launching the EC2 Instance
First, I spun up an EC2 instance to host the Foundry VTT application.

Steps:
1. In the AWS Console, launch a new **EC2 instance**.  
   - Choose a lightweight Linux AMI (e.g., Amazon Linux 2 or Ubuntu).  
   - Instance type: `t3.medium` or larger (Foundry needs some memory and CPU for multiple players).  
2. Configure security groups:  
   - Allow inbound traffic on **port 30000** (Foundry default) or whichever port you configure.  
   - Allow SSH (port 22) for admin access.  
3. Assign an Elastic IP so the server has a stable address.  
4. SSH into the instance and install dependencies:  
   ```bash
   sudo yum update -y
   sudo yum install -y nodejs npm
   ```
5. Upload the Foundry VTT package (after purchasing from Foundry) and install it.

## Configuring Foundry
Once Foundry was installed, I:

- Configured it to listen on the server’s IP address and chosen port.
- Enabled SSL termination through a reverse proxy (Nginx) so I could later hook it up to CloudFront.

## Offloading Static Assets to S3
Foundry generates lots of static assets (maps, images, audio). To make serving these files more efficient, I used S3:

1. Created an S3 bucket (e.g., my-foundry-assets).
2. Configured Foundry’s data path or storage settings to sync uploads to S3 (this can be done using a plugin or a cron job that syncs local Data/ folders to S3).
3. Gave the EC2 instance an IAM role with s3:PutObject and s3:GetObject permissions for the bucket.

## Adding CloudFront in front of S3
To make asset delivery faster worldwide, I put a CloudFront CDN in front of the S3 bucket.

1. Created a new CloudFront distribution with the S3 bucket as the origin.
2. Set the default behavior to allow only GET and HEAD.
3. Enabled caching for static files (images, audio, JS).
4. Requested a free SSL certificate in AWS Certificate Manager (ACM) for my custom domain.
5. Attached the domain and SSL certificate to the distribution.
6. Updated DNS in Route 53 (or my registrar) to point assets.mydomain.com to the CloudFront distribution.

Now, all static files load via https://assets.mydomain.com/... instead of directly from the EC2 instance.

## Summary

With this architecture I was able to optimize for latency and give a better user experience for me and my friends, while still keeping the costs to a minimum.
An EC2 instance runs the game server and handles game logic, user connections and sessions, etc.
S3 hosts many of the static files, which are served by CloudFront and help reduce latency on the bulk of the files that must be transferred to users.
While the costs arent free, they are no bigger than a single dollar. Pretty affordable for me.
