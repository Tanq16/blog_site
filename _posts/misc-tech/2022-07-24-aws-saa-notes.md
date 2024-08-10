---
title: AWS Certified Solutions Architect - Associate Notes
date: 2022-07-24 00:00:00 +0500
categories: [Cloud]
tags: [aws,solutions-architect-associate,course]
---

# AWS Certified Solutions Architect - Associate Notes

The sections here are based on the course by ACloudGuru, and the notes list the most important points I learned overall for the CSAA certification, as well as some based on practice of incorrect mock questions.

## AWS Fundamentals
* Encryption is shared between AWS and customer for the Shared Responsibility Model.
* The Well Architected Framework consists of 5 pillars &rarr;
    * Operational excellence
    * Performance efficiency
    * Security
    * Cost optimization
    * Reliability

## IAM (Identity and Access Management)
* Add an Admin Group and make user accounts for supposed administrators. Add these to the group to make them administrators.
* IAM is universal and not regional.
* IAM users created first have no permissions.
* *Policies* can be assigned to users, groups, roles which are called identity based policies and they can be *managed* (AWS or customer) or *inline* (attached directly to identities).
* Another type of policies is resource based policies which are attached to a resource such as bucket policy
* Resource based policies are inline policies and there are no managed resource based policies.
* If a resource based policy needs to be added to allow a principal from a different account, specifying that principal in the policy is just half the work; an identity based policy must be applied to the principal in the other account to grant access to the resource.
* An IAM role is an identity and a resource which supports a resource based policy.
* Identity based policy on a role can define what access that role has while the resource based policy (role trust policy) defines who can assume that role (access that resource).
* Access Control Lists (ACL) is the only type of policy that does not use JSON format and is supported by only certain services like S3, WAF and VPC.
* Service Control Policies (SCPs) can be applied to any or all entities of accounts within an organization or organizational unit (OU), including the root user in each account.

## S3 (Simple Storage Service)
* S3 scales automatically with demand.
* An S3 URL is of the form &rarr; `https://bucket-name.s3.region.amazonaws.com/key-name` where key-name is basically the filename.
* Versioning cannot be disabled once enabled and a deleted (versioned) object can be restored by removing the delete marker.
* S3 Replication does not replicate existing objects or delete markers by default.
* S3 Object Lock can store objects using a WORM (write once read many) model, which has following modes.
    * Governance mode when specific users can overwrite or delete an object version.
    * Compliance mode when an object version cannot be modified by any user, including root.
* In S3 Object Lock, *Retention Period* dictates when the protections expire, while *Legal Hold* allows protections until someone revokes them.
* WORM in Glacier is via S3 Glacier Vault Lock, which uses compliance mode via a vault lock policy i.e., once locked, the policy cannot be modified.
* S3 is encrypted in transit using TLS while there are 3 modes for server side encryption (SSE) at rest &rarr; 
    * SSE-S3 (uses AES-256) where Amazon handles everything.
    * SSE-KMS (uses KMS to store encryption keys).
    * SSE-C (uses customer managed keys to encrypt data).
* Client Side Encryption (before uploading to S3) is enforced with a bucket policy that denies requests without `x-amz-server-side-encryption` header.
* Using multiple prefixes (path without object name) improves performance on S3.
* Cost-wise S3 storage classes are as follows &rarr; 
    * Standard
    * Intelligent tiering
    * Standard IA
    * One-Zone IA
    * Glacier Instant Retrieval
    * Glacier Flexible Retrieval
    * Glacier Deep Archive
* S3 storage classes have minimum duration limits to keep the objects in storage, which start with 30 days for IA based classes and goes up gradually till Glacier (≥ 120 days).
* If data is needed to be archived for cheap as well as needs to be infrequently accessed rapidly upon request, the most cost effective way is S3 Glacier Instant Retrieval.

## EC2 (Elastic Compute Cloud)
* In a Security Group, all inbound traffic is blocked by default and all outbound traffic is allowed by default.
* Three different networking devices &rarr; 
    * ENI (Elastic Network Interface) is for basic networking, the default option and is lowest in cost.
    * Enhanced Networking is for requirement of high throughput.
    * EFA (Elastic Fabric Adapter) is used to accelerate HPC and ML stuff and for an OS-level bypass.
* Placement Groups are a logical grouping of instances &rarr; 
    * Cluster PG is used for low latency and high throughput but cannot span multiple AZs (both of the remaining ones can).
    * Spread PG is used when instances that need to be on different underlying hardware.
    * Partition PG is used when set of instances are to be put in a given partition for compliance needs or hadoop based stuff.
* Instances can be moved into a PG but they need to be stopped first.
* Spot instances can be blocked from terminating for 1-6 hours using Spot Block.
* Spot Fleet is a collection of Spot instances as well as On-Demand instances (optional).
* AWS Quotas are region specific unless explicitly specified and EC2 instances have a vCPU based limit for On-Demand instances.
* Hot attach means when it's running, warm attach is when it's stopped and cold attach is when the instance is being launched.

## Storage Services
* Volumes are on EBS while Snapshots are on S3.
* Snapshots can be shared to other accounts and in other regions, but they need to be copied to the other region first.
* EBS volume type and its size can be modified on the fly.
* Instance Store Volumes are basically ephemeral storage and provide high random I/O as they are physically attached to the host machines.
* Instance Store Volumes and EBS Volumes will both retain data after a reboot, but Instance Store Volumes cannot be stopped or they will loose data.
* By default both EBS and Instance Store root volumes will be deleted on termination but for EBS, EC2 can be instructed to keep the root volume.
* To encrypt an unencrypted volume &rarr; 
    * Create a snapshot of the root volume, then create a copy of the snapshot while selecting the encrypt option.
    * Then create an AMI from the snapshot and use that to launch new encrypted instances.
* Instances cannot be hibernated for more than 60 days.
* EFS supports many concurrent connections and is on-demand priced for the used storage; it is mainly for Linux.
* In EFS, data is stored across multiple AZs in a region, and can scale up to Petabytes.
* FSx for Windows is a centralized storage for Windows applications.
* FSx for Lustre is for high speed things like HPC; it can also store data directly into S3.
* AWS Backup is used for backing up AWS services like EC2, EBS, EFS, FSx for Lustre, AWS Storage Gateway and FSx for Windows.
* AWS Backup provides central control over backups, across Organizations as well.
* Point in time snapshots in EBS are faster than uploading data to S3 even though snapshots are stored in S3.
* Instance Store Volumes are needed for high performance drive operations like local cache on instances.
* When an AMI is copied from region A into region B, it automatically creates a snapshot in region B because AMIs are based on the underlying snapshots.
* Encryption of data before writing to disk can be done by AWS KMS.
* EBS encryption can encrypt data but only after it is written to disk.

## Databases
* RDS is for OLTP (transactional processing) and not for OLAP (analytical processing), which requires Redshift, etc.
* Read Replica is primarily for scaling read performance and not for DR and can be in the same AZ, cross-AZ or cross-region.
* Read Replicas requires automatic backups to be enabled and multiple read replicas can exist.
* Multi-AZ is only for DR where an exact copy of the DB is in another AZ. If there is a failure, RDS will automatically failover to standby instance.
* When a Multi-AZ RDS is provisioned, it automatically and synchronously replicates data to a read replica in a different AZ. AZ outage, primary DB failure, change of server type, DB OS software patching and manual failover using Reboot are reasons which cause a failover.
* Aurora has a minimum of 2 copies in each AZ and a minimum of 3 AZs.
* Aurora has automatic backups enabled by default.
* DynamoDB is spread across 3 geographically distinct data centers.
* DynamoDB is used for ACID (atomicity, consistency, isolation, durability) requirements across ≥1 tables within a single account and region.
* Throttling in DynamoDB can be solved by turning on DynamoDB AutoScaling.
* DynamoDB On-Demand Backup and Restore takes full backups at any time without any impact on performance and availability.
* DynamoDB Point in Time Recovery protects against accidental writes or deletes and can restore to any point in the last 35 days.
* Dynamo DB is not in-memory and needs DAX to make it so. In-memory usually is done by multipurpose/caching DBs like Redis and DAX.
* For anything greater than a TB in a structured DB with support for complex analytic queries and parallelization, Redshift is best suited, not RDS.
* Warm Standby DR strategy maintains a scaled-down fully functional version of workload but takes a bit of time for failover and scale out, while multi-site strategy is better to instantly failover (full capacity maintained) but costs more too.
* Multi-AZ is within a region so RDS can be used to create a cross-region read replica of database which can be promoted to primary later on.
* In RDS Multi-AZ, if the primary fails, the CNAME is switched from primary DB instance to the secondary.
* Read traffic cannot be directly redirected to a secondary AZ instance of Multi-AZ RDS, instead Read Replicas should be used. Read Replicas get information asynchronously from the database unlike synchronous in Multi-AZ.
* Failover for an RDS Multi-AZ database takes 1-2 minutes and is not instant.
* Redshift Enhanced VPC Routing allows all COPY and UNLOAD traffic between the cluster and data repositories to go through the VPC instead of through the internet, which is the default behavior.

## VPC (Virtual Private Cloud)
* 1 subnet is always in 1 AZ and doesn't span multiple AZs.
* Default NACL comes with a VPC and allows all inbound and outbound traffic. Custom NACLs deny everything by default.
* Each subnet must be associated with a NACL. If it's not done, it will be automatically associated with default NACL.
* A NACL can be associated with many subnets but a subnet can only be associated with 1 NACL at a time.
* VPC endpoints allow connecting to AWS services without leaving AWS internal network. Two types &rarr; interface and gateway. Gateway endpoints only support S3 and DynamoDB.
* Peering connects two VPCs via private IP addresses.
* VPC peering is not transitive in nature; if many VPCs need to be peered with a given VPC (service provider), use PrivateLink which doesn't need peering, route tables, NAT gateways, etc. but needs an NLB on service provider VPC and network interfaces on all other VPCs.
* Setting up Direct Connect is much harder than setting up a VPN as Direct Connect bypasses the Internet and is a dedicated connection that takes a month to get setup.
* AWS Direct Connect bypasses the internet, can provide 10Gbps bandwidth and a dedicated connection to AWS private IP addresses from data center to AWS.
* AWS VPN on the other hand uses the Internet to provide a private connection with encryption.
* Default SG cannot be deleted but the rules can be changed.
* A subnet is classified as public if traffic is routed to an internet gateway.
* Transferring data over Direct Connect does not use encryption in transit, so security needs cannot be met.
* The default [* All Traffic Deny] rule in custom NACLs cannot be modified or removed.
* Setting up a Bastion Host requires its connectivity to the internet as it can't be in a local-only network.

## Domains
* Alias record is an AWS thing and can translate naked domain name or a subdomain to ELB or S3, etc.
* Route53 Traffic Flow is needed for Geoproximity routing.
* Mutivalue Answer allows returning up to 8 healthy records and selecting randomly for serving traffic among those records.

## Load Balancing
* ALBs have listeners which check for connection requests from clients on the protocol and port configured.
* Rules determine how the ALB routes requests to registered targets like EC2 instances.
* NLB provide high performance or features that ALB doesn't provide.
* To get the IP of the client, use the `X-Forwarded-For` header.
* Sticky sessions allow clients to stick to the same EC2 instance for Classic LBs.
* ALBs can also have sticky sessions but only for target groups and not per instance. Could instead have 1 instance per target group if really needed.
* De-registration delay allows the LB to keep connections open to an instance even if the instance becomes unhealthy.
* ALBs can route traffic to targets by inferring routes based on the private IP addresses of the instances.
* WAF can be added to an ALB in a VPC which has Geo-Match conditions to help block access from specific geolocations.
* Both ALB and NLB support dynamic port mapping.

## Monitoring
* CloudWatch can do monitoring in the best way. AWS best practices can be done using AWS Config.
* Basic CloudWatch metrics are gathered every 5 minutes, detailed metrics are gathered every 1 minute (it is costly).
* CloudWatch Logs Insights allows querying logs via SQL-like queries.
* Monitoring activities are the job of CloudWatch whereas CloudTrail is used for logging.
* Memory Utilization is not available by default in CloudWatch Metrics out of the box, but is available in a custom metric along with other similar metrics like Disk Swap, Page File, Disk Space, Log Collection.
* You can now turn on a trail across all regions for your AWS account. CloudTrail will deliver log files from all regions to the Amazon S3 bucket and an optional CloudWatch Logs log group you specified. Additionally, when AWS launches a new region, CloudTrail will create the same trail in the new region.

## Scaling and Availability
* Things should be highly available and cost effective; horizontal scaling is better most of the time.
* Switching between databases is easy in AWS for the SAA exam.
* Putting stuff inside an AMI instead of things like putting them in User Data is better.
* Always put stuff in ≥2 AZs for highly available things or auto-scaling.
* Steady state groups is for instances that cannot be scaled but can recover because min, max and required states are all =1.
* DynamoDB scaling is easily handled by AWS, but comes down to access patterns like predictable workloads can use auto-scaling method and on-demand method for the unpredictable workloads.
* Auto-scaling Scheduled Action or Scheduled Scaling sets the min and max to what the user wants for desired capacity and makes sure that happens when the schedule hits.
* Backlog per instance is a good metric to determine autoscaling of EC2.
* Use Auto-Scaling launch templates over launch configurations as that has more options like versioning.
* Adding Lifecycle Hooks to the Auto Scaling Group puts the instance into a wait state before termination. During this wait state, you can perform custom activities to retrieve critical operational data from a stateful instance. The Default Wait period is 1 hour.

## SQS, SNS and CloudWatch Alarms
* Instant or synchronous workload needs ELBs while asynchronous or acceptable delay workloads need SQS queues.
* SQS can duplicate messages very improbably, but generally it is the fault of misconfiguring visibility timeouts.
* Queues are not bidirectional, so if needed, two queues must be setup.
* Notifications are best delivered by SNS, which can be used with CloudWatch Alarms.
* SQS FIFO Queues can batched to increase performance in multiples from the default 300 messages per second. Example &rarr; batch of 10 messages per operation allows ingesting 3000 messages per second.

## Big Data Services
* Redshift is a relational DB but doesn't replace RDS; Redshift is a single-AZ deployment and needs creation of multiple clusters manually in other AZs.
* EMR is made up of standard EC2 instances so spot instances or RIs can be used to save costs.
* Kinesis Data Streams is for real-time while Kinesis Data Firehose is for near real-time and automatic scaling with ease of use.
* Both SQS and Kinesis can act as queues, but Kinesis can store data for a year (unlike 14 days limit of SQS) and is also real-time.
* Athena is for serverless SQL and can be used to query logs in S3.
* Streaming data can be done using Kinesis Streams but processing those streams needs Kinesis Analytics, which supports standard SQL queries.
* Glue is a serverless ETL tool that can create a schema for data stored in S3 and Athena can then query it.
* Visualizing data can be done using QuickSight but if a third party logging service is needed, then use Elasticsearch.

## Serverless
* S3, Kinesis, EventBridge (CloudWatch Events) are all services that can kick off Lambda functions.
* Lambda functions can have up to 10 GB or RAM and 15 minutes of runtime; CPU scales with the RAM.
* Any AWS API call can be a trigger for an EventBridge rule; this is also faster than checking through CloudTrail.
* ECS only supports AWS and not on-premise.
* When you launch an Amazon ECS container instance, you have the option of passing user data to the instance. The data can be used to perform common automated configuration tasks and even run scripts when the instance boots. For Amazon ECS, the most common use cases for user data are to pass configuration information to the Docker daemon and the Amazon ECS container agent.
* To use Fargate, EKS or ECS must be used.
* Lambda Key Configuration can be used to encrypt the environment variables that can be used to change the function without deploying any code.
Lambda Encryption Helpers make the function secure by allowing encryption of environment variables.

## Security
* Shield protects against layers 3 and 4 attacks only; It is free but advanced is expensive and gives 24/7 support.
* WAF operates at layer 7 and can block access to certain countries or IP addresses.
* GuardDuty uses AI to learn normal behavior in the account and alerts any abnormal behavior.
* GuardDuty monitors CloudTrail logs, VPC Flow logs and DNS logs; EventBridge can be used to trigger Lambda functions to address a GuardDuty alert.
* Macie identifies PII, HPI and PCI data in S3; it can also send alerts to EventBridge.
* S3 private files can be shared using Pre-signed URLs.
* Inspector can perform vulnerability checks on EC2 (host) or VPC (network).
* KMS manages creation and control of encryption keys; it can be started by creation of a CMK (customer master key).
* There are 3 ways to create a CMK &rarr; 
    * AWS generates within KMS HSMs
    * Import key material from customer key management infrastructure and associate with a CMK
    * Use key material stored and generated in an AWS CloudHSM cluster as part of custom key store feature of KMS
* CMK has key policy and is also controllable using IAM policies in combination with the key policy. There are also grants in combination with the key policy, which allows delegation of access as well.
* KMS has shared tenancy of AWS hardware, but CloudHSM is for dedicated physical hardware.
* Rotation of secrets in Secrets Manager rotates them once immediately.
* Parameter Store is used to minimize costs but can only store 10000 parameters.

## Automation
* CFN failures are usually due to unchanged AMIs in a new region, hardcoded parameters, etc. Instead, create a mapping section or store parameters in Parameter Store so there is a unique AMI value per region.
* AWS Elastic Beanstalk can do a whole lot but isn't very robust, for which CFN is used.

## Caching
* CloudFront is the only way to add HTTPS to S3 websites.
* Global Accelerator allows caching of IP addresses by giving two static IP addresses to users.
* Redis is a caching solution and a database solution.
* Redis has more features than Memcached like being a data store and having backups.
* Memcached and DAX are not sources of truth for data.

## Governance
* Centralized logs are always preferred, whether it's into a single bucket or within a single account.
* Any rules to be setup for compliance, can be done through AWS Config.
* Automation Documents can be used within Config to automatically remediate problems.
* Config can be used to see what has changed with a history of such changes within the architecture.
* Use AWS SSO (or with AD and other type of options) for external users and Cognito for internal users.
* Use Managed Microsoft AD when solutions need to be migrated to the cloud and AD Connector if the AD is staying On-Premises.
* Shutting down services or instances to save cost should be always done using Lambda or Automation Documents.
* Trusted Advisor is free, but advanced checks are paid; it is only for auditing and cannot fix things.
* A solution to fixing stuff is using EventBridge and trigger Lambda functions to fix problems.

## Data Migration
* Snowball is used used for TB-scale migration. It also has encryption by default.
* Storage Gateway is run locally as a VM.
* DataSync is an agent based solution and it is only for one time migration unlike Storage Gateway.
* DataSync can send data to EFS and FSx.
* Transfer Family allows legacy applications of (S)FTP(S).
* Database Migration Service is to migrate DB with schema conversion from Oracle or SQL to any other database system.
* Server Migration Service is used to migrate VMs.
