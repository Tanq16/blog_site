---
title: AWS Solutions Architect Associate Notes Part 1
date: 2022-06-12 00:00:00 +0500
categories: [The Cloud, AWS]
tags: [aws,saa,course,notes,iam,s3,cloud,policies]
---

# AWS Fundamentals

| Resources |
| --- |
| [AWS Whitepapers](https://aws.amazon.com/whitepapers/) |

1. *Region* is a location and an *AZ* is a data center (building with servers)
2. An AZ may have multiple data centers, but close to each other
3. A Region contains 2 or more AZs
4. *Edge Locations* are endpoints for AWS used for caching content (generally used by CloudFront) and are around 215 total

AWS sells services to customers. Customers are responsible for their data and configurations, while Amazon is concerned with security of underlying infrastructure. This is also called **Shared Responsibility Model**.

The **Well Architected Framework** consists of 5 pillars &rarr;
* *Operational excellence* → Running and monitoring systems to deliver business value and continually improving processes and procedures
* *Performance efficiency* → Using IT and computing resources efficiently
* *Security* → Protecting information and systems
* *Cost optimization* → Avoiding unnecessary costs
* *Reliability* → Ensuring a workload performs its intended function correctly and consistently when expected to

---

# Identity and Access Management (IAM)

| Resources |
| --- |
| [IAM Policies and Permissions](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies.html) |

1. IAM allows management of **users**, **groups** and **roles** along with their level of access or permissions and IAM operates globally (not in a region)
2. These permissions can allow or deny access to resources and are in the form of a **policy document** which is in JSON format
3. The *root* account is the email used to sign up for AWS and has full administrative access in the account and must always have MFA enabled
4. The *root* account must not be used, instead an admin group must be created and user added to that
5. *Policies* can be assigned to users, groups, roles which are called **identity based policies** and they can be *managed* (AWS or customer) or *inline* (attached directly to identities)
6. Another type of policies is **resource based policies** which are attached to a resource such as bucket policy
7. *Resource based policies are inline policies* and there are *no managed resource based policies*
8. If a resource based policy needs to be added to allow a principal from a different account, specifying that principal in the policy is just half the work; an identity based policy must be applied to the principal in the other account to grant access to the resource
9. IAM service supports only one type of policy called **role trust policy** (attached to a role)
10. An *IAM role is an identity and a resource* which supports a resource based policy
11. Identity based policy on a role can define what access that role has while the resource based policy (role trust policy) defines who can assume that role (access that resource)
12. **Permission boundary** defines the maximum set of permissions that can be granted to an IAM entity by an identity based policy
13. Resource based policies are not affected by permission boundaries
14. **Access Control Lists (ACL)** is the only type of policy that does not use JSON format and is supported by only certain services like S3, WAF and VPC
15. ACLs are service policies that control what principals in other accounts can do, and do not work on principals within the same account
16. Identity based policies cannot be attached to the root user and permission boundary cannot be set for the root user; only ACL and SCPs of an organization affect a root user
17. **Service Control Policies (SCPs)** can be applied to any or all entities of accounts within an organization or organizational unit (OU), including the root user in each account
18. Policies should be applied to groups based on functions and users should be added to groups as a best practice as it can be hard to manage permissions on users
19. IAM federation can be used to set up identity federation using SAML (AD) or OpenID

---

# Simple Storage Service (S3)

## Information

S3 is a managed storage service that can store and retrieve any amount of data. It has a flat file structure and stores data as **objects**, but not OS or database level data. Each object can be between *0 (touch) and 5 terrabytes*. Objects are stored inside **buckets**.

An S3 URL is of the form → `https://bucket-name.s3.region.amazonaws.com/key-name` where key-name is basically the filename. S3 has a key-name to value map and can hold multiple versions of the same object with **versioning** using version ID.

S3 is always spread across multiple devices and facilities for 11-9's durability. S3 also has tiered storage (different storage classes) and lifecycle management.

## Security

S3 allows server side encryption for all objects. ACLs can be attached to objects and policies can be attached to buckets called bucket policies. Therefore, fine-grained access control can be applied using object ACLs but collective controls use bucket policies.

Buckets and all objects within them are private by default and the bucket is marked with **Block all Public Access**. Bucket can be made public using bucket policies and individual objects can be made public using object ACLs. To enable ACLs, the *Block all Public Access* setting must be disabled for the bucket and the *ACLs enabled* setting must be enabled within object ownership as ACLs are disabled when bucket owner is enforced.

Public setting can be used to host a static site from S3 which allows auto-scaling. Static hosting can be enabled from Properties and the index and error HTML documents must be specified.

## Versioning and Lifecycle Management

**Versioning** is great for backup and once enabled can only be suspended not disabled. It can be used with lifecycle rules as well as supports MFA such as for deletion of object versions. The first version of an object has the version ID of `null` whereas newer versions have a random ID.

All versions of objects also have URLs of their own. Public settings don't apply to old versions, so they must be enabled per object version to access them publicly. 

When a version enabled object is deleted, it is not listed but when viewing all versions of objects, the deleted one is visible with a `Delete Marker` type. For a deleted version enabled object, deleting the delete marker object version restores it.

**Lifecycle Management** can be used to move objects among different storage tiers to save money. This can be triggered by *last accessed in number of days*, for example. Lifecycle rules can be applied to specific objects or to the entire bucket.

## S3 Storage Classes

There are 6 classes in S3 &rarr;
1. S3 Standard &rarr; Data is stored in multiple devices across ≥ 3 AZs. It is the default class and is used for frequently access data.
2. S3 Standard IA (Infrequent Access) &rarr; It allows fast access for data that is needed infrequently. It is great for disaster recovery. It has low cost per GB storage but has a per GB retrieval fee.
3. S3 One Zone IA &rarr; It is like Standard IA but data is only stored in a single AZ with 20% less cost overall. It should be used for infrequently accessed non-critical data.
4. Glacier &rarr; It is used for data archiving and has retrieval time between 1 minute to 12 hours. It is very cheap and has a per GB retrieval fee and should be used for data that's needed couple times a year.
5. Glacier Deep Archive &rarr; It is same as Glacier but retrieval time is fixed at 12 hours and should be used for data that is needed < 2 times a year.
6. S3 Intelligent Tiering &rarr; This used machine learning to determine best tier based on access frequency.

S3 gets cheaper as more data is stored.

## Object Encryption & Object and Vault Lock

S3 used TLS for encryption in transit and offers the following for encryption at rest &rarr;
* Server Side Encryption &rarr;
	* S3-managed keys with AES-256 bit encryption
	* Key is managed by AWS KMS 
	* Keys are provided by the customer
	* A file upload to S3 uses a PUT request, which can be used to specify SSE by using a header `x-amz-server-side-encryption` which can have the value as `AES356` or `aws:kms`.
	* Using SSE-KMS automatically calls `GenerateDataKey` in KMS when uploading an object and calls `Decrypt` when downloading it.
* Client Side Encryption &rarr; files are encryption before making into cloud

**S3 Object Lock** can be used to store objects using **WORM (Write Once Read Many)** model. It has a *governance mode* which allows some privileged users to delete and overwrite an object version or change lock settings. In *compliance mode*, not event root user is allowed to delete or overwrite a bucket version.

**Retention Period** can specify an expiration of protection of object versions from overwriting or deletion, unless it is placed on a **Legal Hold**. Legal hold is like retention period but has no expiry until removed by someone who has the `s3:PutObjectLegalHold` permissions.

**Glacier Vault Lock** can be used to enforce compliance controls like WORM on Glacier Vaults. Once locked, policy can no longer be changed for Glacier.

## Performance and Replication

A **Prefix** in S3 for a file `bucket/dir/sub-dir/file` is `dir/sub-dir` i.e., absolute path of objects without the object name and bucket name. In general, S3 has low latency and first byte can be extracted in 100-200 milliseconds. The number of concurrent requests allowed are 3500 for COPY/POST/PUT/DELETE and 5500 for GET/HEAD. Spreading out reads across prefixes can speed up performance. Example &rarr; for 2 prefixes, the number of requests become 11000 GET/HEAD.

KMS has a quota for decryption and key generation calls when SSE-KMS is used. The quota is for concurrent requests per region and is around 5500 requests per second.

**Multipart uploads** allow parallel uploads for segments to increase efficiency and are recommended for files over 100 MB and required for those over 5 GB. **S3 Byte Range Fetch** can be used to parallelize downloads as well as allows downloading specific byte ranges of the objects.

**S3 Replication** can be used to backup data. It used to be called Cross-Region Replication but allows backups to another bucket in any region. Replication requires versioning to be enabled in both source and destination buckets. If making a replica from an existing bucket, the objects are not automatically copied. Also, delete markers are not replicated by default.

IAM automatically creates a new role when enabling replication. Replica bucket's storage tier can be changed from that of the source to save money.

---
