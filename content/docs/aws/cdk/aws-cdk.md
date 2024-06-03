---
weight: 100
title: "AWS CDK"
description: ""
icon: "article"
date: "2024-05-31T08:41:52-05:00"
lastmod: "2024-05-31T08:41:52-05:00"
draft: false
toc: true
---

# Getting Started

## What is the CDK

The AWS Cloud Development Kit (AWS CDK) is an open-source software development framework for defining cloud infrastructure in code and provisioning it through AWS CloudFormation.

The AWS CDK consists of two primary parts:

AWS CDK Construct Library – A collection of pre-written modular and reusable pieces of code, called constructs, that you can use, modify, and integrate to develop your infrastructure quickly. The goal of the AWS CDK Construct Library is to reduce the complexity required to define and integrate AWS services together when building applications on AWS.

AWS CDK Toolkit – A command line tool for interacting with CDK apps. Use the AWS CDK Toolkit to create, manage, and deploy your AWS CDK projects.

The AWS CDK supports TypeScript, JavaScript, Python, Java, C#/.Net, and Go. You can use any of these supported programming languages to define reusable cloud components known as constructs. You compose these together into stacks and apps. Then, you deploy your CDK applications to AWS CloudFormation to provision or update your resources.

## Preparing the Environment

All AWS CDK developers, regardless of preferred language, need Node.js 14.15.0 or later installed. All supported languages use the same backend, which runs on Node.js.

- [ ] [Install Node.js 14.15.0 or later.](https://nodejs.org/en)
- [ ] [Setup an AWS account to use.](https://docs.aws.amazon.com/cdk/v2/guide/getting_started.html#getting_started_account)
- [ ] [Configure Programmatic Access.](https://docs.aws.amazon.com/cdk/v2/guide/getting_started.html#getting_started_auth)
- [ ] [Install the AWS CDK CLI.](https://docs.aws.amazon.com/cdk/v2/guide/getting_started.html#getting_started_install)
- [ ] [Boot Strap Environment.](https://docs.aws.amazon.com/cdk/v2/guide/getting_started.html#getting_started_bootstrap)
- [ ] [Optional Setup.](https://docs.aws.amazon.com/cdk/v2/guide/getting_started.html#getting_started_tools)

## Setup an Account to Use

- [ ] Log in to an existing administrator account or your AWS root user account.
- [ ] Navigate to `IAM Identity Center`.
- [ ] In the navigation pane click users.
- [ ] Create a new user.
- [ ] Assign that user to a group. Create a new group if one doesn't exist.
- [ ] In the navigation pane click AWS Accounts.
- [ ] Select your management account.
- [ ] Assign the group you created to the management account.
- [ ] Create and assign a permission set for that group. 
- [ ] Test your user login.

## Configure Programmatic Access

This section also walks through the process of installing the AWS CLI. 

### Install the AWS CLI.

```
# For Windows Systems
msiexec.exe /i https://awscli.amazonaws.com/AWSCLIV2.msi
```

```
# For Linux Systems (fresh install)
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install

# Updating Install
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install --bin-dir /usr/local/bin --install-dir /usr/local/aws-cli --update
```

Test your AWS CLI Installation.

```shell
aws --version
```

### Configure AWS CLI

```shell
aws configure sso
```

Verify you can login and retrieve your account information.

```shell
aws sso login --profile <profileName>
aws sts get-caller-identity --profile <profileName>
```

### Install the AWS CDK CLI

```shell
npm install -g aws-cdk
```

# Bootstrap your environment

You will need your account number and region.

```shell
aws sts get-caller-identity --profile <profileName>
aws configure get region --profile <profileName>
```

Bootstrap the environment.

```shell
cdk bootstrap aws://<AccountNumber>/<Region> --profile <profileName>
```