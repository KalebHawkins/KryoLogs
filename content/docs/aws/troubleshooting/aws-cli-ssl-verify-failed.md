---
weight: 999
title: "AWS CLI SSL Verify Failed"
description: "Freakin certs man..."
icon: "article"
date: "2024-06-03T10:36:26-05:00"
lastmod: "2024-06-03T10:36:26-05:00"
draft: false
toc: true
---

## Problem

Executing a command like `aws sts get-caller-identity` returns a message similar to the one below.

```shell
SSL validation failed for https://sts.us-east-1.amazonaws.com/ [SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed: self-signed certificate in certificate chain (_ssl.c:1006)
```

## Root Cause

You might be going through a proxy server. To view more information on the cert chain and the error message you can use the `openssl s_client` to peak into what is going on.

```shell
openssl s_client -connect sts.us-east-1.amazonaws.com:443 -showcerts
```

## Resolution

You will need to get the RootCA cert and add it to the AWS CLI's configuration. Once you have your cert you may need to convert it into a `pem` format.

```shell
openssl x509 -inform DER -in certificate.cer -out certificate.pem
```

You can now set the RootCA as your ca bundle. This can be done using the `config` file, `--ca-bundle` command line arg, or the `AWS_CA_BUNDLE` environment variable. See below examples for each of these.

### Using the config file.

```shell
vi ~/.aws/config

ca_bundle = "/path/to/cabundle"
```

### Using Command Line Argument

This will have to be appended to every command.

```shell
aws sts get-caller-identity --ca-bundle "/path/to/cabundle"
```

### Using Environment Variable

This has to be set every session.

```shell
set AWS_CA_BUNDLE="/path/to/cabundle"
```