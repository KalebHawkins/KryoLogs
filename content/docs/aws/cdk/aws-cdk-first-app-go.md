---
weight: 101
title: "Building Your First CDK Application"
description: "Build a simple app that creates a single S3 bucket using Go."
icon: "article"
date: "2024-06-02T20:56:51-05:00"
lastmod: "2024-06-02T20:56:51-05:00"
draft: false
toc: true
---

# Building a CDK Application with Go

In this tutorial we will look at how to build a simple app that creates a single S3 bucket using Go.

For more detail reference the [AWS Documentation](https://docs.aws.amazon.com/cdk/v2/guide/hello_world.html). Their documentation also shows you how to do the same thing in other languages like Python.

## Initialize New Application

```shell
# You can substitute go for whatever language.
cdk init app --language go
# Install the required Go modules
go get
```

## Build the Application

In most programming environments, you build or compile code after making changes. This isn't necessary with the AWS CDK since the CDK CLI will automatically perform this step. However, you can still build manually when you want to catch syntax and type errors. The following is an example:

```shell
go build
```

## List the Stacks in the App

List the stacks in your app to verify the project built correctly. The output of the below command should contain `HelloCdkStack`. If it doesn't try to recreate your project using the steps above.

```shell
cdk ls
```

## Add an Amazon S3 Bucket

Your app contains a single stack at this point. Now we will define an S3 bucket resource within the stack. To do this we will import the `Bucket` L2 construct from the AWS Construct Library.

In the `hello-cdk.go` file look for the comments below:

```go
// The code that defines your stack goes here

// example resource
// queue := awssqs.NewQueue(stack, jsii.String("HelloCdkQueue"), &awssqs.QueueProps{
// 	VisibilityTimeout: awscdk.Duration_Seconds(jsii.Number(300)),
// })
```

This is where you will start writing your code.

We are going to replace the comments with the following.

```go
awss3.NewBucket(stack, jsii.String("MyFirstCDKBucket"), &awss3.BucketProps{
		Versioned: jsii.Bool(true),
})
```

## Synthesize an AWS CloudFormation Template

Synthesize an AWS CloudFormation template for the app, as follows:

```shell
cdk synth
```

This command creates an AWS CloudFormation template for each stack in your app. The CDK CLI will display a YAML formatted version of your template at the command line and save a JSON formatted version in the `cdk.out` directory. 

## Deploy Your Stack

{{< alert context="warning" text="You must perform a one-time bootstrapping of your AWS environment before deployment. For instructions, see [Bootstrap your environment](https://docs.aws.amazon.com/cdk/v2/guide/getting_started.html#getting_started_bootstrap)." />}}

To deploy your CDK stack to AWS Cloud formation you can use the following command:

```shell
cdk deploy
```

{{% alert context="info" %}}
You might receive a message like the one below. 

```
Unable to resolve AWS account to use. It must be either configured when you define your CDK Stack, or through the environment
```

If you do get this message use the following command.

```shell
cdk deploy --profile <profileName>
```
{{% /alert %}}

After running `cdk deploy`, the CDK CLI displays progress information as your stack is deployed. When complete, you can go to the [AWS CloudFormation console](https://us-east-1.console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks?filteringText=&filteringStatus=active&viewNested=true) to view your `HelloCdkStack` stack. You can also go to the Amazon S3 console to view your `MyFirstBucket` resource.

## Modifying Your App

Here we are going to configure the S3 bucket to be automatically deleted when your stack is deleted. To do this we will be changing the bucket's `RemovalPolicy` property. We will also configure the `autoDeleteObjects` property to configure the CDK CLI to delete objects from the bucket before destroying it. By default, AWS CloudFormation does ***NOT*** delete S3 buckets that contain objects.

In your code we will modify the stack to look like this.

```go
 awss3.NewBucket(stack, jsii.String("MyFirstBucket"), &awss3.BucketProps{
		Versioned:         jsii.Bool(true),
		RemovalPolicy:     awscdk.RemovalPolicy_DESTROY,
		AutoDeleteObjects: jsii.Bool(true),
	})
```

At this point the code changes have not made any direct updates to your deployed S3 bucket. The code defines the desired state of your resource. To modify your deployed resource use the CDK CLI to synthesize the desired state into a new AWS CloudFormation template.

To see the changes, use the `cdk diff` command. 

```shell
cdk diff --profile <profileName>
cdk synth
cdk deploy # If you are happy with those changes.
```

## 🔥Burn It All Down🔥

This command will destroy the stack and all associated resources.

```shell
cdk destroy --profile <profileName>
```