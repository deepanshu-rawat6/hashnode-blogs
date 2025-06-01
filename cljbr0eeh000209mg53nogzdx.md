---
title: "Experimenting with Bash Scripts"
seoTitle: "Secure EC2 Instances with Bash Scripts"
seoDescription: "Learn how to use bash scripts to enhance the security of your EC2 instances, by using Bastion hosts and Key Rotations"
datePublished: Sun Jun 25 2023 18:13:26 GMT+0000 (Coordinated Universal Time)
cuid: cljbr0eeh000209mg53nogzdx
slug: experimenting-with-bash-scripts
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1687712525995/d6bfa1ca-38ea-473e-9b21-0bc2ddcf9f1e.jpeg
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1687713032109/448efbcd-57db-46ec-b5ae-b444dc4368a0.jpeg
tags: aws, security, bash, devops, wemakedevs

---

Heyy folks, this blog contains some scripts to enhance your knowledge of networking and security while working on AWS EC2. The scripts are as follows:

* **Bastion Connect**
    
* **SSH Key Rotation**
    

Before going into the scripts, let's see the setup of the VPC, Subnets, Route Tables, Internet Gateway, and EC2 instances.

Disclaimer: It is advised to read this blog in **dark mode**, to enhance the overall experience

---

# Setup

## VPC

A [VPC](https://docs.aws.amazon.com/vpc/latest/userguide/what-is-amazon-vpc.html) with IPv4 CIDR block of `10.0.0.0/16`, with all other settings set to default.

## Subnets

### Public Subnet

A public [subnet](https://docs.aws.amazon.com/vpc/latest/userguide/configure-subnets.html) within the same VPC, with default availability zone and IPv4 CIDR block of `10.0.0.0/24`. Additionally, the **auto-assign IP address** is enabled.

### Private Subnet

A private subnet within the same VPC, with default availability zone and IPv4 CIDR block of `10.0.1.0/24`. Here, the **auto-assign public IP** is disabled.

## Custom Route Table

A custom [route table](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Route_Tables.html) is created within the same VPC. Now, added routes to the table as:

* For IPv4 traffic added `0.0.0.0/0` and select the internet gateway as the target
    
* Added both subnets in the subnet association tab
    

## Internet Gateway

An [Internet Gateway](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Internet_Gateway.html) is created and attached to the VPC, and also attached to the custom route table and the VPC.

## EC2 Instances

#### Public Instance

An [EC2 instance](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/concepts.html), within the same VPC, attaching the public subnet to it.

#### Private Instance

An EC2 instance, within the same VPC, and attaching the private subnet to it.

## Common configurations

* t2.micro [instance type](https://www.amazonaws.cn/en/ec2/instance-types/)
    
* Ubuntu 22.04.02 LTS [AMI](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AMIs.html)
    
* 8 GiB [gp3](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-ec2-volume.html) volume
    
* Same [key pair](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/create-key-pairs.html) for both instances
    

## Some observations

On running the public instance, we can see that it has a public IP address as well as a private IP address, but the private instance has a private IP address only.

## Architecture

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687714870307/0b40add5-9ab1-40d2-8195-eb5732d5f177.png align="center")

So, here's the architecture of the VPC. The public instance is accessible from the Internet, and the private instance is not accessible from the Internet.

Now, enough boring configurations, let's get started with scripts✨

%[https://media.giphy.com/media/f6z5TkrTIBZILYOd1t/giphy.gif] 

---

# Bastion Connect Script

This script is used to connect to the private instance from the public instance. The script is as follows:

```bash
#!/bin/bash

# Checking whether the KEY_PATH env var is set
if [ -z "$KEY_PATH" ] then
    echo "KEY_PATH env var is expected"
    exit 5
fi

# Taking arguments, in variables
public_instance_ip="$1"
private_instance_ip="$2"
command="$3"

# Checking whether the public_instance_ip is set
if [ -z "$public_instance_ip" ] then
    echo "Please provide bastion IP address"
    exit 5
fi

# Checking whether the private_instance_ip was provided
if [ -z "$private_instance_ip" ] then
    # This command connects to the bastion host
   	ssh -i "$KEY_PATH" ubuntu@"$public_instance_ip"

# Checking whether the command was provided
elif [ -z "$command" ] then
    # This command connects to the private instance, with help of bastion host
    ssh -i "$KEY_PATH" ubuntu@"$public_instance_ip" ssh -t -t -i "~/new_key" ubuntu@"$private_instance_ip"
else
    # This command connects to the private instance, with help of bastion host, and executes the command, and then exits
	ssh -i "$KEY_PATH" ubuntu@"$public_instance_ip" ssh -t -t -i "~/new_key" ubuntu@"$private_instance_ip" "$command"
fi
```

Here, the public instance is the bastion host, and the private instance is the instance to which we want to connect to directly which is not accessible from the internet, without the hassle of connecting to a public instance first and then connecting to the private instance.

***Note:*** The environment variable `KEY_PATH` is the path to the private key to the public instance(bastion host). Also, before running do not forget to add execute permission to the script. Use this command to add [permission](https://www.geeksforgeeks.org/permissions-in-linux/) to the script:

```bash
chmod +x <name of script>
```

The script takes **three** arguments:

* Public instance IP address
    
* Private instance IP address
    
* Command to be executed on the private instance
    

Some examples of the script:

1. If the environment variable `KEY_PATH` is not set, then the script will exit with code 5.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687715248845/5f2a9708-5449-4ea6-b21b-dbc0f4e5d3aa.png align="center")
    
2. If the bastion host's IP address is not provided then the script will exit with code 5.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687715270474/555504ed-fa7d-411d-87a8-f81d96898c1e.png align="center")
    
3. Connecting to the Bastion host
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687715314687/f238e835-622e-46cb-b701-6f842c8fd5e1.png align="center")
    
4. Connecting to the private instance, with the help of the Bastion host
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687715335382/2552f7b7-ace7-4a87-8552-6fa6a1e9a931.png align="center")
    
5. Executing a command on the private instance, then exiting to the local machine.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687715358956/1eef75d4-8009-40b8-bd3e-5a09e0f49768.png align="center")
    
    Since we can connect to both public and private instances, now let's focus on the security of our instances, and we can do so by rotating our ssh keys.
    

---

# SSH Key Rotation script

This script is used to rotate the SSH keys of the private instance. The script is as follows:

```bash
#!/bin/bash

if [ -z "$1" ] then
        echo "Please provide IP address"
        exit 1
fi

private_instance_ip="$1"
user="ubuntu"

# Generating the new key
ssh-keygen -t rsa -f new_key_temp -N ""

# Copy the newly generated key to the private instance
scp -i ./new_key ./new_key_temp.pub "$user@$private_instance_ip":~

# Adding the new_key.pub to the authorized keys of private instance
ssh -i ./new_key "$user@$private_instance_ip" "cp -f ~/new_key_temp.pub ~/.ssh/authorized_keys"

# temp key to new key
cp -f ./new_key_temp ./new_key
cp -f ./new_key_temp.pub ./new_key.pub

# To see if the new key works
ssh -i ./new_key "$user@$private_instance_ip"
```

To execute this script we need to be SSHed(I don't know whether its a term or not 😂) into our Bastion host first

Here, the script generates a new key by using [`rsa algorithm`](https://www.simplilearn.com/tutorials/cryptography-tutorial/rsa-algorithm), and then copies the public key to the private instance, and then adds the public key to the authorized keys of the private instance. Then, the script replaces the old keys with new keys. Finally, the script checks whether the new key works or not.

The script takes **one** argument:

* Private instance IP address
    

Some examples of the script:

1. If the private instance IP address is not provided then the script will exit with code 1.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687715799624/1edb14a8-7481-43ac-9b01-9fa4a2063e34.png align="center")
    
2. Rotating the SSH keys of the private instance, and then connecting to the private instance with the new key, to check whether the script works or not.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687715817020/d6ed78af-6190-4dc5-83bc-977ef0f8f7a9.png align="center")
    

After we are done rotating our keys, we verify the working of the `bastion_connect` script

***Verification:*** The script works, as we can see that the new key works, and we can connect to the private instance with the new key. Also, our `bastion_connect` script works as well, as we can connect to the private instance with the new key, with the help of the bastion host.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1687715840804/8af750d6-efd1-4bd9-957d-36f42e68442a.png align="center")

---

# Good Practices

1. While creating instances always create a **security group**, and add the security group to the instance, instead of adding the rules directly to the instance.
    
2. If working in a group account, always set **stop protection** and **termination protection** to the instances, so that no one can stop or terminate the instance by mistake.
    
3. Always use a **bastion host** to connect to the private instances, as it is not accessible from the internet, and it is more secure.
    
4. Always **rotate the SSH keys** of the private instances, so that the private instances are more secure.
    
5. Most importantly, never forget to stop your instance after you are done with your work, otherwise a huge bill will be waiting for you at the end of the month👀
    

---

# Final Words

So this was it for this blog everyone, this was a long but surely informative blog on how to properly use AWS EC2 service.

If you liked this blog, a like would be appreciated, and do let me know areas of improvement. Also, if you want me to blog about some other services of AWS or DevOps, do let me know in the comments or you can reach me on Twitter.

See you on the next blog✨