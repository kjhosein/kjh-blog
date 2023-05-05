---
title: "How to Launch a Linux AWS EC2 Instance and set the Hostname to a Tag's value"
date: 2015-10-01T10:00:00-04:00
draft: false
---

This howto describes how to launch an AWS EC2 instance and in particular, set the hostname to the same value as the Name tag of the instance.

These notes can also be referenced to create a command-line based method, an API-based method or even be done via IaC (e.g., Terraform).

- Log into the AWS Console and navigate to [EC2 service -> Instances](https://console.aws.amazon.com/ec2/v2/home?#Instances:).
- Click the _Launch Instances_ button and select a Linux AMI. e.g., Ubuntu, Red Hat, Amazon Linux, etc.
- Ensure you use an Instance Profile with an IAM Role that allows reading tags (i.e. `ec2 describe-tags`). The managed _ReadOnlyAccess_ IAM role allows this, but you may want to narrow this down by writing your own more restricted custom policy.
- Entering an FQDN name for the server should create a tag with a Key of “Name”, and a Value of the intended hostname (FQDN) of this machine.
- Here's where the magic happens. Paste the following script into the user-data section:

```
#!/bin/bash

REGION=$(curl -s http://169.254.169.254/latest/dynamic/instance-identity/document | awk -F\" '/region/ {print     $4}')
SERVER=$(aws --region ${REGION} ec2 describe-tags --filters "Name=resource-id,Values=$(curl -s http://169.254.169.254/latest/meta-data/instance-id)" "Name=key,Values=Name" --query 'Tags[*].Value' --output text)
PRIVATE_IP=$(curl -s http://169.254.169.254/latest/meta-data/local-ipv4)

sed -i "s/^\(HOSTNAME\s*=\s*\).*$/\1$SERVER/" /etc/sysconfig/network
echo "$PRIVATE_IP $SERVER" >> /etc/hosts
hostname $SERVER
```

This Bash script will run upon boot-up of the virtual server instance. What it does is:

1. Query the instance's meta-data for its region, and stuff that into an environment variable called `REGION`.
2. Use that _REGION_ value in an AWS CLI query to get the value of the _Name_ tag. This tag contains your desired FQDN for the server.
3. Query the instance's meta-data for its private IP address and place that into an environment variable called `PRIVATE_IP`.
4. Replace any _HOSTNAME_ lines in the `/etc/sysconfig/network` file with the correct hostname value.
5. Add a line into `/etc/hosts` with the hostname and private IP (i.e. a local DNS entry of the server).
6. Rename the host using the `hostname` command.

NOTE: the server requires the AWS CLI to already be installed, so if this fails for your particular Linux distribution, include commands at the top of the Bash script to [install the AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html).

> Update: 2020: Note that if you're launching instances and are requiring [IMDSv2](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/configuring-instance-metadata-service.html), you'll need to consider and incorporate authentication into your meta-data queries.
