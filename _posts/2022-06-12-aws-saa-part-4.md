---
title: AWS Solutions Architect Associate Notes Part 4
date: 2022-06-12 00:00:00 +0500
categories: [The Cloud, AWS]
tags: [aws,saa,course,notes,cloud,big-data,redshift,emr,serverless,lambda,fargate,cloudtrail,guarddty,acm,secrets-manager,presigned-url,presigned-cookie,migration,aws-organizations,scp,iac,cfn,caching]
---

# Big Data

- **Redshift** is a fully managed data warehouse that can hold up to 16 PB of relational DB data; In a Redshift cluster, a master node receives the queries and divides them to send to the other nodes for processing; Redshift can only be setup in a single AZ.
- **ETL (Extract Transform Load)** is a process of transforming large data to give it meaning and analyze it. **Elastic Map Reduce (EMR)** helps with the ETL process.
- EMR is a big data managed platform that allows processing of data using Spark, Hive, HBase, Flink, Hudi and Presto; EMR cluster can be run inside EC2 or EKS; The data is processed and stored in S3.
- **Kinesis** allows ingesting, processing and analyzing real-time streaming data; It makes it easy to load and analyze the large volumes of data entering AWS.
	- *Kinesis Streams* &rarr;
		- They retain the data that enters it for 1-7 days
		- Once inside Kinesis Streams, the data is contained within shards
		- Customer need to define consumers and handle scaling, so Amazon built Firehose
	- *Kinesis Firehose* &rarr;
		- It is the easiest way to load streaming data into data stores and analytics tools
		- No persistent storage, so when data enters, it is processed and sent elsewhere like S3, Redshift, Elasticsearch and Splunk
	- *Kinesis Analytics* &rarr; 
		- It works with both Kinesis Streams and Firehose and can analyze data on the fly
		- It also sends data elsewhere like Firehose
- **Athena** &rarr; It is an interactive query service that can analyze data in S3 using direct SQL queries without loading the data into a DB i.e., serverless kinds
- **AWS Glue** &rarr; It is a serverless data integration service that can perform ETL workloads without managing underlying servers and tries to replace EMR
- **QuickSight** is a fully managed BI data visualization service like Tableau that can have dashboards and moves away from the spreadsheet like stuff
- **ElasticSearch** is a fully managed ElasticSearch service that can quickly search over stored data and analyze the retrieved data; commonly used as part of an ElasticSearch, Logstash, Kibana (ELK) stack

---

# Serverless Architecture

* In SAA, answers which have serverless are more favored over other options
* **Lambda** is a serverless compute service that can run code without provisioning or managing underlying servers
* To build a Lambda function &rarr;
	* Pick an available runtime like Python, NodeJS; containers can also be run inside
	* Attach an appropriate role if the function needs to make an AWS API call
	* Configure network (VPC+SGs) is function needs to talk to other services
	* Define resources like memory and CPU
	* Define a trigger that can alert the function to execute
* To manage 100s of containers, **ECS (Elastic Container Service)** and **EKS (Elastic Kubernetes Service)** are used
	* ECS integrates natively with LBs
	* ECS cannot run on-premise workloads but EKS can
* **Fargate** is a serverless compute engine for running containers that work with ECS/EKS
* **Amazon EventBridge** or **CloudWatch Events** is a serverless bus that allows passing events from source to an endpoint by creating a rule for it
	* Define Pattern for rule (can be cron or based on an API call)
	* Select Event Bus mainly for AWS based events
	* Select targets like trigger a lambda function, post to SQS or SNS topic
	* Using Event Bridge is faster than analyzing CloudTrail

---

# Security

* **CloudTrail** can log information like source IP, user and account info for API calls
* Actions within a managed service such as RDP or SSH are not logged and need agents
* Metadata of API calls, Identity of API, Time, Source IP, Req/Response of calls are logged
* **AWS Shield** is free DDoS protection for all customers on ELB, CloudFront and Route 53
* **AWS Shield Advanced** protects against larger and more sophisticated attacks
	* Active flow based monitoring of network traffic and real time DDoS notifications
	* 24/7 access to DDoS Response Team (DRT)
	* Protection against high bill usage in times of a DDoS
	* It costs $3000 per month
* **WAF** allows monitoring of HTTP(S) requests forwarded to CloudFront or ELB
* WAF can allow all except specified, block all except specified, or count requests that match properties specified
* **GuardDuty** is a threat detection service that uses ML to look for malicious behavior
* GuardDuty receives feedback from third parties like ProofPoint, CrowdStrike and AWS Security about known malicious domains and IPs
* GuardDuty monitors CloudTrail Logs, VPC Flow Logs and DNS logs
* GuardDuty can be centralized across multiple accounts
* GuardDuty takes 7-14 days to set a baseline and is free for 30 days
* **Macie** does automated analysis of data in S3 using ML and pattern matching to discover sensitive data, and also alerts for unencrypted buckets
* Macie sends events to EventBridge, so logs can be integrated with SIEM solutions
* **Inspector** is an automated security assessment service that can look at network (open routes to VPC) or hosts (vulnerabilities in EC2 instances like outdated OS, CVEs)
* **KMS (Key Management Service)** gives control over lifecycle and permissions of keys
* **Customer Master Key (CMK)** is a logical representation of a master key that contains key material to encrypt/decrypt data
* A **Hardware Security Module (HSM)** is a physical computing device with a crypto processor chip that safeguards and manages digital keys and performs encryption and decryption functions
* Three ways to generate a CMK &rarr;
	* AWS generates key material with HSMs and manages using KMS
	* Import key material from client key management infra and associate with CMK
	* Key material generated and used in an AWS CloudHSM cluster as part of the custom key store in KMS
* CloudHSM is customer-dedicated hardware and does not have automatic key rotation, though KMS has but is shared tenancy
* **Secrets Manager** stores, encrypts and rotates secrets or database credentials
* With automatic rotation selected, Secrets Manager rotates the secret once immediately
* **Parameter Store** is a capability of AWS Systems Manager for secure hierarchical storage for configuration data and secrets management
* Parameter Store is free but no automatic rotation and only 10000 values can be stored
* An S3 object owner can share the object with others by creating a **presigned URL**
* Presigned URL grants time limited permissions to download objects
* **Presigned Cookies** are provided to people to store on browser when multiple restricted files need to be shared
* **AWS Certificate Manager (ACM)** allows creating, managing and deploying SSL certs
* ACM is free but money is needed when services use ACM like ELB
* ACM also handles auto-renewal of certs and auto-update in services like ELB

---

# Automation

* **IaC (Infrastructure as Code)** tools like **CloudFormation (CFN)** by AWS and **Terraform** by Hashicorp can be used to provision infrastructure and resources in cloud
* **Elastic Beanstalk** is an all in one service to build web applications (PaaS)
* **Systems Manager** can easily patch, update, manage and configure EC2 instances as well as on-premise infrastructure (paid for on-premise otherwise free)
* If CFN finds an error, it rolls back to the last known good state

---

# Caching

* **CloudFront** is a CDN and a global caching service that can front several applications
* CloudFront is the only way to add HTTPS to an S3-hosted static site
* **ElastiCache** is a managed version of open source Memcached and Redis
* **Memcached** is simple DB caching, while **Redis** can also serve as a standalone multi-AZ DB with failover
* **DynamoDB Accelerator (DAX)** is a 1000x performance increase caching solution
* IP Fixing issue and Global Accelerator &rarr;
	* If an application is load balanced, users of it cache the IP of the LB
	* If the LB goes off road, and after recovery, new LB has new IP, then user cannot reach
	* To solve, **Global Accelerator** sits in between and masks LB IP with two static IPs
	* Global Accelerator uses AWS edge locations, so connections are faster

---

# Governance

* **AWS Organizations** allows governance of multiple AWS accounts
* Organizations can programmatically create/delete accounts, provide consolidated billing and can share RIs across all accounts
* **Service Control Policies (SCPs)** can be used to limit permissions for the accounts
* **Resource Access Manager (RAM)** is a free service that can share resources with other accounts in an organization
* Default VPCs cannot be shared
* RAM is used when resources are shared in the same region, otherwise VPC sharing
* **AWS Config** allows inventory management for resources in use and their current state
* AWS Config provides querying, enforcement of fixes and alerts and logging of events
* **AWS Directory Service** is a fully managed version of Active Directory with types &rarr;
	* *Managed Microsoft AD* &rarr; entire AD suite built on AWS
	- *AD Connector* &rarr; creates a tunnel between AWS and on-premise AD so users get an endpoint to authenticate against while leaving the actual users and data on-premise
	- *Simple AD* &rarr; standalone directory powered by Linux Samba AD
- **AWS Cost Explorer** provides billing reports and querying
- **AWS Budgets** allows budget planning and alerting
- **Trusted Advisor** is a fully managed best practice auditing tool

---

# Migration

* Moving data in and out of the cloud over internet is very slow
* Moving over Direct Connect is faster but expensive for excess data
* **Snow Family** allows moving PBs of data to AWS &rarr;
	* *Snowcone* &rarr; smallest; have 8 TB capacity, 4 GB memory and 2 vCPUs; it has an IoT sensor and is good for edge computing
	* *Snowball Edge* &rarr; It has 48-81 TB storage with varying amounts of memory and CPU/GPU based on storage, compute and GPU flavors
	* *Snowmobile* &rarr; the biggest and it's a semi-truck that can move 100 PB of data; allows exabyte-scale data migration
* **Storage Gateway** is a hybrid cloud storage that merges on-premise and cloud resources for a one time migration to/from the cloud
* Storage Gateway solutions are VMs which means they need to be spun up inside on-premise architecture
* **File Gateway** &rarr; NFS or SMB mount to back up to S3
* **Volume Gateway** &rarr; it is an ISCSI mount to backup on-premise VM drives
* **Tape Gateway** &rarr; it can backup physical tapes to S3, Glacier or Glacier Deep Archive
* **AWS DataSync** is an agent based solution for migrating large on-premise storage once to AWS from NFS or SMB shares; used for one time sync whereas Storage Gateway is continuous
* **AWS Transfer Family** allows moving data in and out of S3 or EFS using SFTP, FTP(S)
* **Migration Hub** &rarr; 
	* *Server Migration Service (SMS)* &rarr; takes in VMWare vSphere images and uploads to AWS, where VMDKs are converted into EBS snapshots (S3) and AMIs are created to launch an equivalent EC2 instances
	* *Database Migration Service (DMS)* &rarr; similar things as SMS but for databases; takes old databases from Oracle or MySQL or something on-premise, or EC2, or RDS and runs it through *AWS Schema Conversion* Tool to store it into an Aurora DB

---
