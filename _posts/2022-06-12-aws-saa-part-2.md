---
title: AWS Solutions Architect Associate Notes Part 2
date: 2022-06-12 00:00:00 +0500
categories: [The Cloud, AWS]
tags: [aws,saa,course,notes,ec2,vpc,cloud,imds,efs,ebs,snapshot]
---

# Elastic Compute Cloud (EC2)

| Resources |
| --- |
| [Granting permissions to applications in EC2 instances](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_use_switch-role-ec2.html) |
| [Using Instance Profiles](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_use_switch-role-ec2_instance-profiles.html) |
| [IMDSv2](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/configuring-instance-metadata-service.html) |

## Overview

EC2 is a pay for what you use VM on AWS' servers which have 4 pricing options &rarr;

1. **On-Demand** &rarr;
	* Pay by the second or hour based on instance type
	* Low cost and no upfront payment, used for flexible workloads like test servers
	* Unpredictable workloads for small businesses
2. **Reserved** &rarr;
	* 1-3 year contract with up to 72% discount on hourly rate
	* Best with longer terms of upfront payment
	* Used for predictable and steady workloads
	* Types of Reserved Instances (RIs) &rarr;
		* Standard RI &rarr; 72% discount on On-Demand price
		* Convertible RI &rarr; have option of changing to another class of EC2 of ≥ value, 54% discount
		* Scheduled RI &rarr; this has a schedule associated so that instance is launched only during that time frame, example is like an early morning tally server
	* RIs operate within a region that cannot be changed
3. **Spot** &rarr;
	* Unused capacity in EC2 which gives 90% discount
	* User defines price at which instance is required
	* If instance is not available at that price, it is decommissioned
	* Used for workloads with flexible start and end times
4. **Dedicated** &rarr;
	* Most expensive option that can be used to conform to regulatory compliance or for specific licensing needs where licenses are not portable
	* They have an hourly rate and offer 70% discount if reserved

## Permissions, Bootstrap Scripts and IMDS

EC2 instances can be secured with **Security Groups** which are stateful. They are placed into subnets where NACLs can also have an effect. Permissions are granted to EC2 instances using roles which can be attached and detached from instance without stopping them. By default all inbound traffic is blocked and all outbound traffic is allowed.

An application within EC2 is abstracted from AWS due to virtualization, so to provide a role for EC2 applications to use, an extra step of creation of an **Instance Profile** and attaching it to an instance. The role that needs to be granted is added to the *Instance Profile* so that EC2 applications can assume that.

When the console is used to create a role and assign that to an instance, IAM automatically creates an instance profile with the same name as the role. When using the API or CLI, first an instance profile should be created, then a role be added to it and then it be associated with an instance.

**Bootstrap** script is one that runs when an instance first starts and it runs with root level privileges. The execution script can be placed inside the `User Data` section. An instance also has a server running locally called the **Instance Metadata Service (IMDS)** which can give information like public and private IP, hostname, etc. From within an EC2 instance, it can be retrieved using &rarr;

```bash
curl http://169.254.169.254/latest/meta-data/local-ipv4
```

Changing `meta-data` to `user-data` also allows querying the user data. The IMDS or IMDSv1 which is the default version doesn't have any authentication or cryptographic protection. 

IMDSv1 is a request/response method while IMDSv2 is a session oriented method. For IMDSv2, first a token must be retrieved as follows &rarr;

```bash
TOKEN=`curl -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 21600"`
```

Then, the request to retrieve metadata can be made as follows &rarr; 

```bash
curl -H "X-aws-ec2-metadata-token: $TOKEN" -v http://169.254.169.254/latest/meta-data/
```

## Networking & Placement Groups

EC2 allows attaching 3 types of virtual networking cards &rarr;

- **ENI (Elastic Network Interface)** &rarr; Basic day to day networking and this is the default
- **EN (Enhanced Networking)** &rarr; Uses *Single Root I/O Virtualization (SR-IOV)* to provide high performance networking
- **EFA (Elastic Fabric Adapter)** &rarr; Accelerates High Performance Computing and Machine Learning applications.

There are three types of **placement groups** &rarr; 
- **Cluster** &rarr; Group of instances within a single AZ
- **Spread** &rarr; Group of instances each on distinct hardware and should be used for applications that need critical instances to be separate (primary, secondary)
- **Partition** &rarr; Every partition PG has a set of racks and each rack has a network and power source; no two partitions share the same racks, i.e., isolation of hardware failure

A cluster PG cannot span multiple AZs but a spread and partition PG can. An existing instance can be moved into a PG but it must be stopped before that.

---

# EBS & EFS

## Elastic Block Storage (EBS)

EBS volumes are virtual storage drives that can be attached to EC2 instances. It is scalable and replicated within a single AZ. Its capacity and type can be changed without downtime. Types of volumes are as follows &rarr;
* **General Purpose SSD (gp2)** &rarr; default storage option
* **Provisioned IOPS SSD (io1 and io2)** &rarr; used for high I/O workloads
* **Throughput Optimized HDD (st1)** &rarr; used for throughput-intensive workloads like big data and is an HDD with lower cost; it cannot be a boot volume
* **Cold HDD (sc1)** &rarr; Lowest cost HDD option mainly for static data; it cannot be a boot volume

## Volumes & Snapshots

- A minimum of 1 volume per EC2 instance must exist, called the root device volume.
- Volumes are always in the same AZ as the instance to which it is attached.
- **Snapshots** are stored on S3 and are a copy of a volume at a point in time.
- Snapshots are incremental i.e., snap 1 may be huge but snap 2 will only have the changes since snap 1.
- Snapshots only capture data written to EBS which doesn't include caches, for which an instance must be stopped before snap.
- The snapshot of an encrypted EBS volume is also encrypted.
- Copying an unencrypted snapshot allows encryption.
- Snapshots can be shared only in the region they were created. To share to other regions, they need to be copied to the destination region and then shared.
- EBS encryption uses AES-256 bit with KMS or CMK.

To encrypt an unencrypted volume, create a snapshot, create a copy of the snapshot and select the encrypt option, create an AMI from the encrypted snapshot and use that AMI to launch new encrypted instance.

The root volume is terminated when an instance is terminated, but that data can be saved if needed. When it is hibernated, the OS suspends to disk (root volume) and all volumes are persisted, with their IDs intact when the instance wakes from hibernation. Instances cannot be hibernated for more than 60 days.

## Elastic File Storage (EFS)

EFS is a managed network file system that can be mounted to EC2 instances. It works with instances in multiple AZs, but it is expensive. It is a great for content management and web servers.

EFS uses NFSv4 protocol and is compatible with Linux instances only. It uses encryption at rest with KMS. Performance characteristics can be set while creating an EFS &rarr; 
- General Purpose &rarr; for things like CMS, web servers, etc.
- Max I/O &rarr; for big data, media processing, etc.

EFS has storage tiers and lifecycle management, and allows moving data from one tier to another after X number of days. The tiers are similar to S3 &rarr; Standard and Infrequently Accessed.

**FSx for Windows File Server** provides a fully managed Microsoft Windows file system for easily moving Windows-based applications to AWS. **FSx for Lustre** is a fully managed file system optimized for compute intensive workloads such as ML and HPC, media processing, etc. It can store data directly on to S3.

## AMIs

**Amazon Machine Image (AMI)** provides a template to launch an instance. All AMIs are backed by either &rarr; 
- EBS &rarr; instance root device launched from AMI is a volume created from EBS snapshot
- Instance store &rarr; instance root device launched from AMI is a volume created from a template stored in S3

Instance Store Volumes (aka ephemeral storage) loose all data if the underlying host fails or shuts down. However, a reboot can be done without loss. They delete upon instance deletion. EBS volumes are only deleted upon instance termination, however, root volume can be persisted after termination by enabling that option.

---

# Virtual Private Cloud (VPC)

## Overview

VPC is like a virtual data center, which can have several subnets and other things. The smallest subnet can be `/28` and the largest is `/16`.

* When a VPC is created, a Route Table, a Network ACL, an SG and a Router are created.
* Then public subnets and entities with SGs can be created.
* An Internet Gateway can be attached to allow instances in public subnet to become Internet accessible.
* A Virtual Private Gateway could also be attached to the Router in the VPC to establish a VPN connection from the data center in the private subnet.
* The default VPC is user friendly so all subnets have a route to the internet. Each EC2 instance has both a public and private IP address.
* Custom VPCs are fully customizable and take time to setup.
* A subnet can only be in one AZ and cannot span multiple AZs.
* When creating a subnet, the `x.x.x.0` IP is the network address, `x.x.x.1` is the VPC router, `x.x.x.2` is the DNS server, `x.x.x.3` is reserved for future functionality and the `x.x.x.255` IP is for broadcast address.
* To make a subnet public, Auto-assign Public IP addresses must be turned on.
* Internet Gateway can be added and is detached by default. It can then be attached to the VPC. There is only one IG per VPC.

There is always a main route table associated with a VPC. All new subnets created are associated with a default route table. For this reason the default route table must not be open to public internet otherwise all new subnets would have a route out to the internet.

Therefore, a new route table must be created. To add a route out to the internet, add `0.0.0.0/0` with the target being the internet gateway, as a route to the new route table. Then edit subnet associations within the new route table and add the subnet needed to be made public.

- NAT Gateways can be used to allow internet access to private subnets in a VPC. A NAT Gateway needs to be provisioned in the public subnet to allow access to internet.
- NAT Gateways are redundant inside the AZ and is automatically patched by AWS
- It is not associated with SGs and is automatically assigned a Public IP address.
- For a private subnet to communicate with the internet, a route needs to be added to the route table associated with it. Usually, this route table is going to be the main route table, where a route can be added to `0.0.0.0/0` should go to the new NAT Gateway.

## Security

- SGs are the last line of defense
- The order of controls is routers, NACLs and then SGs
- SGs are stateful i.e., outbound of a port allows the communication reverting for that irrespective of the inbound rules
- NACLs are the first line of defense that act as a firewall for one or more subnets within a VPC. It can be setup with rules similar to SGs to add another layer of security to the VPC
- A VPC comes with default NACL that allows all inbound and outbound traffic by default
- Custom NACLs can be created which deny all traffic by default until rules are added
- Each subnet must be associated with a NACL, if nothing else, the default NACL is used
- IP addresses can be blocked using NACLs but not using SGs (single using `/32` CIDR)
- A subnet can only be associated with 1 NACL but a NACL can be associated with multiple subnets
- A NACL has a numeric list of rules that are evaluated in order (first allow/deny works)
- NACLs are stateless so separate inbound and outbound allow/deny rules

## VPC Endpoints

They enable users to privately connect a VPC to *supported AWS services* and *VPC endpoint services powered by PrivateLink* without the need of an internet gateway, NAT device, VPN or Direct Connect. So traffic between the VPC and the service is on Amazon's network backbone and therefore, the VPC does not need a public IP address.

Endpoints are virtual devices that are scaled horizontally. They have no bandwidth constraints. Writing stuff into S3 from EC2 for example does not need a NAT gateway as it will be overloaded for no reason at all and therefore the VPC endpoints are great.

There are two types of endpoints &rarr; 
- Interface &rarr; ENI with a private IP that serves as an entry point for traffic to a supported service
- Gateway &rarr; similar to NAT GWs (virtual device) supporting connections to S3 and DynamoDB

## VPC Connections

To talk to each other, VPCs need **peering**. *Peering* allows connecting one VPC with another via a direct network route using private IP addresses. VPCs can be peered with other AWS accounts or VPCs in the same account.

Peering follows a star configuration and there are no transitive peer connections. Peering can be done between regions as well. The peered VPCs cannot have overlapping IP addresses.

Sharing connections between applications and some services is not always best if done through VPC peering as all the applications will be accessible through that link and can have security implications. Another way to do this is using **PrivateLink**.

- It is the best way to expose a service VPC to tens or thousands of customer VPCs
- It doesn't require VPC peering or route tables or NAT gateways or internet gateways
- It needs a Network Load Balancer on the service VPC and an ENI on the customer VPC

A **VPN CloudHub** is useful if a business has multiple sites and each has its own VPN connection that need to connected together. It has a hub and spoke model like VPC peering. It has low cost and is easy to manage. It operates over the public internet and all traffic between the customer gateway and AWS VPN CloudHub is encrypted.

**Direct Connect** is used to simplify establishing a dedicated network connection from on premise to AWS. This has a direct line with AWS so, a more stable connection. This can reduce cost and the overall bandwidth. They are of two types &rarr; 

- Dedicated &rarr; physical ethernet connection associated with a single customer
- Hosted &rarr; provided by an AWS Direct Connect partner

VPNs allow private communication but it still traverses the public internet to get the data delivered. It is secure but slow. Direct Connect is fast, secure, reliable and can carry massive throughput. 

**Transit gateway** connects VPCs and on premise networks through a central hub. This simplifies complex peering relationships and acts as a cloud router so each new connection is only made once.
- Everything connected to the transit gateway can talk to each other
- It also has a hub and spoke model
- It supports IP multicast and use of route tables to limit how VPCs talk to each other
- It works on a regional basis but can be across multiple regions
- It can also be used across multiple accounts using RAM (Resource Access Manager)

---
