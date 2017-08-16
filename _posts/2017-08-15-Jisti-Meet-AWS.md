---
title: "Install Jisti-Meet on ubuntu 16.04 with AWS EC2"
layout: post
date: 2017-08-15
headerImage: false
tag:
- jisti
- aws
- ec2
- ubuntu
blog: true
author: gdf
description: "The beginner guide for installing Jisti-Meet on ubuntu 16.04 with AWS EC2 service"
---

# Create an EC2 instance on AWS

![IMAGE](/assets/images/posts/aws_ubuntu_ec2.png)
## Security groups setting
1. Go to the [EC2](https://console.aws.amazon.com/ec2) portal of AWS
2. Navigate to the `Security Groups` inside `NETWORK & SECURITY`
3. Select the corresponding security group and click `Actions -> Edit inbound rules`
4. Add the `TCP 443`, `TCP 4443`, and `UDP 10000-2000` ports to the rules like the following
5. ![IMAGE](/assets/images/posts/security_inbound.png)
6. (Option) Link your domain to the EC2 instance
    - Naviate to the `Elastic IPs` inside `NETWORK & SECURITY`
    - Choose `Allocate new address`
    - Selecet the elastic ip that you just create and right click to `associate address`
    - Choose `Instance` and select your instance and private ip
    - Add a new DNS record on you domain point to the new `Elastic IP` (ex: xx.xx.xx.xx)

# Install the Jitsi Meet

## A. Use `ssh` to login the EC2 instance that you just created
[http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AccessingInstancesLinux.html](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AccessingInstancesLinux.html)
## B. Step to install the Jitsi Meet
Follow the instruction on [Jitsi Meet quick install guide](https://github.com/jitsi/jitsi-meet/blob/master/doc/quick-install.md) or just type the following
```
sudo su
echo 'deb https://download.jitsi.org stable/' >> /etc/apt/sources.list.d/jitsi-stable.list
wget -qO -  https://download.jitsi.org/jitsi-key.gpg.key | apt-key add -
apt-get update
apt-get -y install jitsi-meet
```
> Note: During installation, you will have to input your hostname and choose the way to installing ssl cerificate. For your hostname, if you link your domain in the previous step 6, enter your domain name (ex: talk.gdf.name). Otherwise, enter the public DNS of your EC2 instance or just the IPv4 public IP address. For ssl cerification，if you know nothing about this, just select the self cerfication option.

## C. (Option) Certificate you self-signed SSL
1. Download the [file](https://github.com/jitsi/jitsi-meet/blob/master/resources/install-letsencrypt-cert.sh) to your EC2 instance
2. Add execution permission to this file by `sudo chmod +x install-letsencrypt-cert.sh`
3. Run the file `./install-letsencrypt-cert.sh`

