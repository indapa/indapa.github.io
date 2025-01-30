---
layout: post
title: "My first AWS Lambda function"
date: 2025-01-23
author: "Amit Indap"
tags:
  - AWS
---

Over the past few months I've become a bit more familiar with various [Amazon Web Services](https://aws.amazon.com/) (AWS) offerings,
incuding [EC2](https://aws.amazon.com/ec2/), [S3](https://aws.amazon.com/s3/), and [Batch](https://aws.amazon.com/batch/). I've heard about [AWS Lambda](https://docs.aws.amazon.com/lambda/latest/dg/with-s3-example.html), but never had a chance to try it out. So I decided to work
through this [tutorial](https://docs.aws.amazon.com/lambda/latest/dg/with-s3-example.html) to create my first Lambda function.

AWS Lambda is a serverless computing service that lets you run code without provisioning or managing servers.
You pay only for the compute time you consume - there is no charge when your code is not running.
With Lambda, you can run code for virtually any type of application or backend service - all with zero administration.
Just upload your code and Lambda takes care of everything required to run and scale your code with high availability.

In the tutorial, the user creates a Lambda function that is triggered adding an object to an S3 bucket, and a Python function 
outputs the object type to CloudWatch logs. The tutorial is very well written and easy to follow:

![Lambda tutorial](../images/aws_lambda.png)

As with any new skill, you have to learn to walk before you run. I'm hoping to use Lambda for my own projects very soon. 
Stay tuned for more updates!


