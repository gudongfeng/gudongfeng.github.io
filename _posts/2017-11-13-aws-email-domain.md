---
title: "Setup serverless email using SES S3 & Lambda with you personal domain"
layout: post
date: 2017-11-13
headerImage: false
tag:
- aws
- s3
- ses
- lambda
blog: true
author: gdf
description: "This tutorial tells you how to setup your personal email address (personal@mydomain.com)"
---

# What for?

Let's assume you have a personal domain (mydomain.com). You want to create an email address with you personal domain on it, for example contact@mydomain.com, and forward all the email that send to contact@mydomain.com to your personal email address (personal@outlook.com). This tutorial suits you best. 

# Setup the domains in SES management console
1. Verify your domain according to the [link](http://docs.aws.amazon.com/ses/latest/DeveloperGuide/receiving-email-getting-started-verify.html).
2. Set up a receipt rule according to the [link](http://docs.aws.amazon.com/ses/latest/DeveloperGuide/receiving-email-getting-started-receipt-rule.html).
   > Take a note of the **Bucket Name** in section 5 for later usage.
3. Login [SES console](https://console.aws.amazon.com/ses).
    - Choose `Email addresses` on the left panel.
    - Select `Verify a New Email Address`.
    - Type in the email address that you want to forward to (in my case is `gudongfeng@outlook.com`).
    - Go through all the steps and verify your email.

# Set the Lambda action for SES
- Download the file [index.js](https://raw.githubusercontent.com/arithmetric/aws-lambda-ses-forwarder/master/index.js).
- Modify the values in the `defaultConfig` to specify the S3 buckget and object prefix for locating emails stored by SES
    In my case:
    {% highlight javascript %}
    ...
    fromEmail: "contact@gdf.name",
    
    subjectPrefix: "",
    emailBucket: "contact-email",
    //emailKeyPrefix: "emailsPrefix/",
    emailKeyPrefix: "",
    forwardMapping: {
      "contact@gdf.name": [
        "gudongfeng@outlook.com"
      ]
    }
    ...
    {% endhighlight %}

- Login AWS [Lambda](https://console.aws.amazon.com/lambda).
   - Cretae a new function.
   - Click `Author from scratch`.
   - Name the function `SesForwarder` and optionally give it a description. Ensure Runtime is set to Node.js 4.3 or 6.10.
   - For the Lambda function code, either copy and paste the contents of `index.js` into the inline code editor or zip the contents of the repository and upload them directly or via S3.
   - Ensure Handler is set to `index.handler`.
   - For Execution role, choose `Create a custom role`. In the popup, give the role a name (e.g., LambdaSesForwarder). Configure the role policy to the following, change the `<your-s3-bucket-name>` accordingly:
    {% highlight ruby %}
    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Effect": "Allow",
                "Action": [
                    "logs:CreateLogGroup",
                    "logs:CreateLogStream",
                    "logs:PutLogEvents"
                ],
                "Resource": "arn:aws:logs:*:*:*"
            },
            {
                "Effect": "Allow",
                "Action": "ses:SendRawEmail",
                "Resource": "*"
            },
            {
                "Effect": "Allow",
                "Action": [
                    "s3:GetObject",
                    "s3:PutObject"
                ],
                "Resource": "arn:aws:s3:::<your-s3-bucket-name>/*"
            }
        ]
    }
    {% endhighlight %}

   - Memory can be left at 128 MB, but set Timeout to 10 seconds to be safe. The task usually takes about 30 MB and a few seconds. After testing the task, you may be able to reduce the Timeout limit.

# S3 bucket policy settings
Login S3 and choose the bucket you just created.
   - Click the `Permissions` on the top menu.
   - Click `Bucket Policy`.
   - Make sure your policy is like following:

    {% highlight ruby %}
    {
        "Version": "2012-10-17",
        "Statement": [
          {
              "Sid": "AllowSESPuts-<sid-number>",
              "Effect": "Allow",
              "Principal": {
                "Service": "ses.amazonaws.com"
              },
              "Action": "s3:PutObject",
              "Resource": "arn:aws:s3:::<your-s3-bucket-name>/*",
              "Condition": {
                "StringEquals": {
                    "aws:Referer": "<your-aws-account-id>"
                }
              }
          }
        ]
    }
    {% endhighlight %}