---
layout: post
title: "Hosting a Static Website on AWS S3"
date: 2024-11-01
categories: [AWS, S3, Web Development]
tags: [AWS, S3, Static Website, Tutorial]
---

In this post, we'll walk through the steps to host a static website using Amazon S3 (Simple Storage Service). S3 is an excellent option for static site hosting due to its reliability, scalability, and cost-effectiveness.

## Step 1: Create an S3 Bucket

1. Log in to your AWS Management Console.
2. Navigate to the **S3** service.
3. Click on **Create bucket**.
4. Enter a unique bucket name (e.g., `your-website-name`) and choose your preferred region.
5. Uncheck **Block all public access** to allow public access to your website. Acknowledge the warning.
6. Click **Create bucket**.

## Step 2: Upload Your Website Files

1. Open your newly created bucket.
2. Click on **Upload** and select the files you want to host. This typically includes HTML, CSS, JavaScript, images, etc.
3. Once uploaded, ensure all files are accessible.

## Step 3: Set Bucket Policy for Public Access

1. Go to the **Permissions** tab in your S3 bucket.
2. Scroll down to **Bucket policy** and add the following policy, replacing `your-bucket-name` with your actual bucket name:

   ```json
   {
     "Version": "2012-10-17",
     "Statement": [
       {
         "Sid": "PublicReadGetObject",
         "Effect": "Allow",
         "Principal": "*",
         "Action": "s3:GetObject",
         "Resource": "arn:aws:s3:::your-bucket-name/*"
       }
     ]
   }
   ```

## Step 4: Enable Static Website Hosting

1. Go to the "Properties" tab in your S3 bucket.
2. Scroll down to "Static website hosting" and select "Enable."
3. Choose "Host a static website."
4. Set the index document (e.g., index.html).
5. Optionally, specify an error document (e.g., error.html).
5. Save the changes.

## Step 5: Access Your Website
You’ll see a URL provided in the "Static website hosting" section `(e.g., http://your-bucket-name.s3-website-ap-southeast-1.amazonaws.com)`.
This URL is now the public URL for your static website.

![Image](/assets/images/StaticWebSiteOKResult.png)

In case the main website fails it redirects to an error page.

![Image](/assets/images/StaticWebSiteErrorResult.png)

Resources: [Github](https://github.com/mrcpngn/hoststaticwebsite-aws-s3)
