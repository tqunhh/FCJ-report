---
title: "Image Processing - Lambda, Rekognition, SQS, SNS, SES"
date: 2025-12-08
weight: 4
chapter: false
pre: "<b>5.4. </b>"
---

#### Overview

In the Travel Guide project, the automated image processing system includes the following key components:

- **Lambda Content Moderation** - Content moderation for images
- **Lambda Detect Labels** - Automatic label detection
- **SQS Queue** - Asynchronous processing
- **SNS Topic & SES** - Email notifications

#### Lambda Functions Architecture

![Full Lambda Functions](/images/5-Workshop/5.4-image-processing/Full_lamdba_Function.png)

#### Processing Flow

1. User uploads image → S3
2. S3 Event → SQS Queue
3. Lambda Content Moderation checks content
4. If valid → Lambda Detect Labels adds tags
5. If violated → SNS + SES sends alert email

#### Content

- [Lambda Content Moderation](5.4.1-content-moderation/)
- [Lambda Detect Labels](5.4.2-detect-labels/)
- [SQS Pipeline](5.4.3-sqs-pipeline/)
- [SNS & SES Notification](5.4.4-sns-ses-notification/)
