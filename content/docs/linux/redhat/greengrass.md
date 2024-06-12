---
weight: 999
title: "Installing AWS Greengrass"
description: "Installing AWS Green Grass on Red Hat Enterprise Linux 9"
icon: "article"
date: "2024-06-12T14:47:56-05:00"
lastmod: "2024-06-12T14:47:56-05:00"
draft: false
toc: true
---

## Provisioning

Ensure that the BIOS, NIC, and GPU have the latest firmware installed. 

Disable any power saving features in the BIOS including hibernation/sleep. 
Also depending on the GPU/CPU different combinations of drivers, Cuda, cudnn and tensorrt needs to be installed. Below is an example for a g4dn ec2 instance.

## Update Linux Librarys and Install Dependencies

```shell
sudo dnf update -y
sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm
sudo dnf install wget openssl-devel libffi-devel bzip2-devel sqlite-devel python3.9 python3-pip kernel-headers -y
sudo reboot
```

## Install NVIDIA Drivers

* [NVIDIA Drivers](https://www.nvidia.com/en-us/drivers/unix/)

```shell
aws s3 cp --recursive s3://ec2-linux-nvidia-drivers/latest/ .
# Replace `{{Version}}` with your downloaded driver version.
sudo CC=/usr/bin/gcc10-cc ./NVIDIA-Linux-x86_64-{{Version}}-grid-aws.run  --kernel-source-path /usr/src/kernels/5.10.165-143.735.amzn2.x86_64/ 
# Go through the prompts 
sudo reboot
```

## Verify installation

```shell
nvidia-smi
```

## Install Nvidia Cuda 

* [Cuda Documentation](https://docs.nvidia.com/cuda/archive/10.2/cuda-installation-guide-linux/index.html#choose-installation-method)

```shell
wget https://developer.download.nvidia.com/compute/cuda/12.5.0/local_installers/cuda-repo-rhel9-12-5-local-12.5.0_555.42.02-1.x86_64.rpm

sudo rpm -i cuda-repo-rhel9-12-5-local-12.5.0_555.42.02-1.x86_64.rpm
sudo dnf clean all
sudo dnf -y install cuda-toolkit-12-5
```

## Install Cudnn

* [Cudnn Documentation](https://developer.nvidia.com/cudnn)

```shell
sudo curl -fsSL https://developer.download.nvidia.com/compute/cuda/repos/rhel9/x86_64/cuda-rhel9.12.4.repo -o /etc/yum.repos.d/cuda-rhel9.repo

sudo rpm -i cudnn-local-repo-rhel9-9.2.0-1.0-1.x86_64.rpm
sudo dnf clean all
sudo dnf -y install cudnn
```

## Install TensorRT

You will need an NVIDIA account to access the downloads you can sign up at [TensorRT](https://developer.nvidia.com/nvidia-tensorrt-download). 

```shell
wget https://developer.nvidia.com/downloads/compute/machine-learning/tensorrt/10.0.1/local_repo/nv-tensorrt-local-repo-rhel9-10.0.1-cuda-12.4-1.0-1.x86_64.rpm

sudo rpm -i nv-tensorrt-local-repo-rhel9-10.0.1-cuda-12.4-1.0-1.x86_64.rpm
sudo dnf install tensorrt -y
```

## Install Greengrass Dependencies

```shell
sudo pip3 install dlr==1.10.0 grpcio==1.43.0 grpcio-tools==1.43.0 numpy==1.22.2 protobuf==3.19.4 scikit-learn==1.0.2 scipy==1.7.3 Pillow==9.0.1 aws-embedded-metrics==1.0.7 awsiotsdk==1.10.0 warlock==1.3.3 jsonschema==3.2.0 torch==1.12.1+cu102

sudo pip3 install torchvision==0.13.1+cu102 --extra-index-url https://download.pytorch.org/whl/cu102

sudo pip3 install kornia_moons 

# Overwrite (or create) the following file /usr/local/lib/python3.9/site-packages/dlr/counter/ccm_config.json 
sudo bash -c 'echo {"enable_phone_home" : false} > /usr/local/lib/python3.9/site-packages/dlr/counter/ccm_config.json'

# Give access to everyone to that file 
sudo chmod 777 /usr/local/lib/python3.9/site-packages/dlr/counter/ccm_config.json

# Check that your /etc/sudoers file gives the user permission to run sudo as other groups. The permission for the user in /etc/sudoers should look like the following example.
sudo cat /etc/sudoers | grep root
# > root    ALL=(ALL)       ALL
```

## Install Amazon Corretto

* [Amazon Corretto Documentation](https://docs.aws.amazon.com/corretto/latest/corretto-17-ug/generic-linux-install.html)

```shell
sudo rpm --import https://yum.corretto.aws/corretto.key
sudo curl -L -o /etc/yum.repos.d/corretto.repo https://yum.corretto.aws/corretto.repo

sudo yum install -y java-17-amazon-corretto-devel
```

```shell
java -version
# openjdk version "17.0.11" 2024-04-16 LTS
# OpenJDK Runtime Environment Corretto-17.0.11.9.1 (build 17.0.11+9-LTS)
# OpenJDK 64-Bit Server VM Corretto-17.0.11.9.1 (build 17.0.11+9-LTS, mixed mode, sharing)
```

## Install Greengrass

Use the `QIS` tool to create the curl command to install Greengrass.

## Install Aauthbind

```shell
sudo rpm -Uvh https://s3.amazonaws.com/aaronsilber/public/authbind-2.1.1-0.1.x86_64.rpm
sudo touch /etc/authbind/byport/80 

## Replace <ggc_user>:<ggc_group> with the appropriate user and group.
sudo chown <ggc_user>:<ggc_group> /etc/authbind/byport/80 
sudo chmod 770 /etc/authbind/byport/80 
```