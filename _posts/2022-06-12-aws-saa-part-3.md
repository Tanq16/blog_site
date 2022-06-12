---
title: AWS Solutions Architect Associate Notes Part 3
date: 2022-06-12 00:00:00 +0500
categories: [The Cloud, AWS]
tags: [aws,saa,course,notes,cloud,sns,sqs,auto-scaling,route53,routing,load-balancer,elb,cloudwatch,api-gateway]
---

# Route53

## Overview

There are several types of DNS records &rarr; 

1. **SOA** &rarr; start of authority; contains information about the server that supplied data for the zone, administrator of the zone, current version of data file and default TTL
2. **NS** &rarr; name server records; used by the TLD servers to direct traffic to the content DNS server that contains authoritative DNS records
3. **A** &rarr; most common record used to lookup the IP address
4. **CNAME** &rarr; canonical name; used to resolve one domain name to another like `m.youtube.com` to `youtube.com`
5. **Alias records** &rarr; like CNAMEs; used to map resource record sets in hosted zones to load balancers, CloudFront distributions, or S3 buckets; Example mapping `elb1234.elb.amazonaws.com` to `example.com`; this is an *AWS only* record, not a general DNS concept

CNAMEs cannot be used for naked domain names (zone apex record). Example, cannot have a CNAME for `tanishqtop.com`. However, Alias records can be used for a naked domain name since they can map into individual services.

## Routing Policies

1. **Simple Routing Policy** &rarr; 1 record with multiple IPs returned to user in a random order
2. **Weighted Routing Policy** &rarr; used to split traffic based on different weights, like 10% to one and rest to others
	*Health Checks* can be set on individual record sets or instances or LBs, etc. For a record set (say 10% is weight A and rest is weight B), if a failure occurs, the health check will fail and all traffic will be sent to the non-failed one. Notifications using SNS can be setup in case of a failed Health Check.
3. **Failover Routing Policy** &rarr; used when there is an active/passive setup like a primary site in `us-east-1` and secondary in `ap-southeast-2`. Failure in active setup causes automatic redirection to the secondary site.
4. **Geolocation Routing Policy** &rarr; lets the customer choose where traffic will be sent based on location of end users.
5. **Geo-proximity Routing Policy** &rarr; used to build a routing location that uses a combination of geographic location, latency and availability to route traffic. Apart from geographic location, an optional value called *bias* can be used to route more or less traffic to a given resource by specifying the value. To use this, Route 53 traffic flow must be setup.
6. **Latency Routing Policy** &rarr; allows routing traffic based on lowest latency for the end users.
7. **Multi-value Answer Routing Policy** &rarr; allows returning multiple IPs in a DNS response as well as allows to check the health of all records and return only healthy records.

---

# Elastic Load Balancing

## Overview

ELB automatically distributes incoming application traffic across targets like EC2 instances. This can be done across multiple AZs. There are 4 types &rarr; 

- **Application Load Balancer (Intelligent)** &rarr; application aware; best for HTTP(S) traffic
- **Network Load Balancer (Performance)** &rarr; layer 4; can handle millions of requests per second with ultra-low latencies
- **Classic Load Balancer (Test/Dev)** &rarr; legacy LBs for HTTP(S) applications
- **Gateway Load Balancer (Third Party)** &rarr; supports connection with third party virtual appliances and uses the GENEVE protocol

All load balancers can be configured with **health checks**. This sends periodic requests to the instances registered with the load balancers to test their status. Healthy instances have a status of `InService`, while unhealthy ones are `OutOfService`. 

## Application Load Balancer

After ALB receives a request, it evaluates the **listener rules** and selects a target from the **target group** for the rule action. **Listeners** check for connection requests as per protocol and port configured.

Rules can determine how the ALB routes to its targets. Each rule has a *priority*, one or more *actions* and one or more *conditions*. When the conditions for a rule are met then the actions are performed. A default rule must be defined for each listener.

Each target group routes requests to one or more registered targets like EC2 instances, using the protocol and port number specified.

When a user browses to a URL which hits Route53 (and then ALB), the URL is hosted in servers in the one AZ while some other URL paths like images which need to hit other EC2 instances in another target group might be in another AZ.

ALB is intelligent and can look at paths if the **Path Patterns** are enabled to make intelligent routing decisions. To use an HTTPS listener, at least one SSL/TLS server certificate must be deployed on the ALB. The ALB uses this to terminate frontend connection and decrypts the requests before sending them to targets.

## Network Load Balancer

When a request is received, the NLB selects a target from the target group for the default rule. It attempts to open a TCP connection to the target on the port specified in the listener configuration.

The listener has no rules, unlike ALB so, there is no intelligent routing. Supported protocols are TCP, TLS, UDP and TCP_UDP. All ports are supported.

A TLS listener can be used to offload the encryption and decryption to the NLB so the applications load is reduced. For the TLS listener, exactly one SSL/TLS server certificate must be deployed.

## Classic Load Balancer

These can load balance HTTP(S) applications and use layer 7 specific features like `X-Forwarded-For` header and *sticky sessions*. Strict layer 4 load balancing can also be used for applications that rely purely on TCP.

When traffic is sent from a load balancer, the server access logs contain the IP of the proxy or the LB only. To see the original IP of the client, the `X-Forwarded-For` request header is used. The port is also sent in the `X-Forwarded-For-Port` header.

If the application stops responding, the Classic LB responds with a 504 or Gateway Timeout error i.e., application has issues which could be web server layer or DB layer.

Classic LBs route each request independently to registered EC2 instances with the smallest load. **Sticky sessions** allow binding a user’s session to a specific EC2 instance. It is useful for cases when the instance needs to write to local storage. The issue with scaling is that if the instance goes off-road, the request will not automatically route to other instances without disabling sticky sessions.

**De-Registration Delay** or **Connection Draining** for Classic LB allows LBs to keep existing connections of the EC2 instances that are de-registered or become unhealthy, open. This allows LB to complete in-flight requests.

---

# Monitoring with CloudWatch

CloudWatch is a monitoring and observability platform for insights into AWS for multiple levels of applications. It can do &rarr; 

1. **System level Metrics** &rarr; managed services have more metrics out of the box
2. **Application Metrics** &rarr; obtained by installing the CloudWatch agent that pulls information from EC2 instances like is the Apache server running
3. **Alarms** &rarr; alert when something goes wrong

Metrics are of two kinds &rarr; 

- *Default* &rarr; out of the box for managed services like CPU utilization, network throughput
- *Custom* &rarr; need to be provided by the CloudWatch agent like EC2 memory utilization, EBS storage capacity

There are no default alarms, all need to be created. Monitoring for less number of services can have logs shifted to S3. But with more services it can be hard. CloudWatch Logs allows monitoring, storing and accessing log files from a variety of different sources.

Three terms for CloudWatch Logs &rarr;

1. *Log Event* &rarr; the record of what happens and a timestamp with data
2. *Log Stream* &rarr; a collection of log events from the same source like from an EC2 instance
3. *Log Group* &rarr; a collection of log streams like aggregate Apache logs across all hosts

- CloudWatch Logs Insights allows querying all logs using a SQL like interactive solution
- CloudWatch agent installation on EC2 with `yum install amazon-cloudwatch-agent -y`
- CloudWatch can also be used for On-Premise systems
- CloudWatch Logs is *NOT* real time, but is generally a go-to solution everywhere
- Metrics are delivered every 5 minutes for free; with fee, can be every minute
- All AWS services integrate with CloudWatch Logs

---

# Scalability

## Overview

Horizontal Scaling is when multiple instances are needed while Vertical Scaling is when the characteristics of a single instance are increased.

A launch template has all needed settings for building an EC2 instance like a snapshot of the wizard settings.

- **Templates** are more than just auto-scaling as they support versioning and more granularity (all options from the wizard)
- **Configurations** are used only for auto-scaling and are immutable, being limited in number of configuration options (not all from the wizard)

## EC2 AutoScaling

An **Auto Scaling Group** has a collection of EC2 instances which are scaled and managed. Steps for Auto Scaling are as follows &rarr; 

- Define the Template &rarr; launch templates or launch configurations
- Networking & Instance Type&rarr; Pick networking option and spot, reserved or on-demand instances; also define at least 2 AZs
- Add ELB &rarr; Register instances behind an LB; deregister when taken down; happens automatically if the ASG is set to interact with the LB (correspond with LB health check)
- Scaling policies &rarr; define minimum, maximum and desired capacity
- Notifications &rarr; SNS to inform of scaling events

Some auto scaling restrictions are as follows &rarr; 

- Minimum instances must be at least 2 &rarr; if set to 0 with 1 instance running, low utilization would terminate that instance
- Maximum &rarr; set at a few instances more than actual max
- Desired Capacity &rarr; number of instances needed at the moment; AS respects this most

## AutoScaling Policies

During scale out, instances are not instantly available so warmup period comes in. Terminating doesn’t need as much time so scale in can be done sparingly but scale out conservatively.

Warm up prevents instances from being placed behind the ELB while starting and failing health check and in turn being terminated. Cool down pauses AS for 5 minutes to avoid runaway scaling events and this happens both at launch and termination.

Types of scaling are as follows &rarr; 

- **Reactive Scaling** &rarr; under load, scale out, otherwise scale in
- **Scheduled Scaling** &rarr; for predictable workloads so resources are available before they are actually needed
- **Predictive Scaling** &rarr; ML and heuristics updates every 24 hours to create a 48 hour schedule and balance between reactive and scheduled methods
- **Steady State Auto Scaling** &rarr; min, max and desired capacity are all `1`, when multiple instance copies are not possible so, at failure, the instance is migrated among the selected AZs to keep it available

## Scaling Databases

There are 4 types of scaling for RDS &rarr; 

- Vertical Scaling &rarr; resizing can improve performance
- Scaling Storage &rarr; only goes up, except Aurora, which AWS manages for size
- Read Replicas &rarr; kinda scaling by sending read requests to read-only replicas
- Aurora Serverless &rarr; offloads scaling to AWS and used for unpredictable workloads

Handling capacity in DynamoDB can be done as follows &rarr; 

- Provisioned Model &rarr; setup AS by defining min, max and target utilization for predictive workloads
- On-Demand Model &rarr; sporadic workloads where read/write capacity can go up or down without warning; this is expensive
- The models can be switched once every 24 hours

---

# Decoupling Workflows

## Overview

**Tight Coupling** &rarr; a frontend instance and a backend instance mean a single point of failure at both places, which is bad and solved by *loose coupling*

**Loose Coupling** &rarr; an ELB acting as frontend distributing to a fleet of instances with another ELB distributing to a fleet of backend instances

**SQS** is a fully managed message queue service that allows decoupling and scaling micro-services, distributed systems, etc. **SNS** is a fully managed messaging service for application to application (A2A) as well as application to person (A2P) notifications.

**API Gateway** is a fully managed service that allows developers to create, publish, design, monitor and secure APIs at any scale publicly.

## SQS

- **Poll Based Messaging** &rarr; producer writes messages to a queue and consumer can consume from the same queue whenever needed
- SQS allows asynchronous processing of work and is not bidirectional
- Delivery Delay &rarr; after writing a message to queue, it is hidden for this period of time; default value is 0 but can go up to 15 mins
- Message Size &rarr; up to 256 KB of any text in any format (json, yaml, whatever)
- Encryption &rarr; encrypted by default in transit, but not rest (can be done if needed)
- Message Retention &rarr; 4 days by default, but can be between 1 minute and 14 days; after this window, messages are purged
- **Short (default) polling** is when the consumer connects to queue, asks for messages, and disconnects but, this burns CPU cycles and API calls
- **Long polling** is when consumer waits after connecting and no response till there is actually a message
- Queue Depth &rarr; a potential trigger for Auto Scaling
- **Visibility timeouts** is a mechanism where messages marked for delivery from the queue are given a time frame (30 seconds default) to be fully received by a reader; they are invisible to other readers during that time; if message is not fully processed till then, it becomes visible again (not purged from queue)

## Ordered Messages & Dead-Letter Queues

- **Standard SQS Queues** offer best-effort ordering (not strict) with possible duplication, but can have almost unlimited transactions per second
- **FIFO Queues** guarantee ordering without duplications but can only handle 300 messages per second; also expensive

FIFO queues use some additional fields per message &rarr; 

- *Message group ID* &rarr; ordering maintained for messages with the same group ID
- *Message deduplication ID* &rarr; token used for deduplication within an interval (if the ID is seen again within the interval, it will not be sent to the consumer)

If a producer sends a message to a queue but made an error, the consumer will consume and fail the process with the message destroyed (or keep on reappearing if visibility timeout is set). **Dead-Letter Queue** can be set up in this case; it's another queue where messages can be sidelined into if specified number of retries (*maximum receives*) is reached.

DLQ must be set inside the main queue’s creation process so it must be created before. The receive count for a message is preserved and reflected in the DLQ as well. SNS topics and alarms can be set on DLQ size.

## SNS

Push-Based messaging as opposed to Poll-Based messaging &rarr; consumer is immediately given the message. SNS is proactive notifications and can be used to alert.

- When message is published in a topic, it is sent out to **subscribers**
- An SNS message can be sent to multiple SQS queue at the same time
- Message size &rarr; Same as SQS i.e., up to 256 KB of text of any size
- DLQ support &rarr; An SQS queue subscribed to an SNS topic
- SNS sidelines and does not retry if delivery fails, with the exception of HTTP(S)
- FIFO or Standard &rarr; FIFO only supports SQS as a subscriber
- Encryption &rarr; encryption in transit by default, can be configured for rest
- Access Policy &rarr; A resource policy can be added to a topic (who or what can publish stuff for a topic)

## API Gateway

API Gateway is a fully managed service that is a safe front door (allow outside to interact with) to an application. It protects endpoints by attaching a WAF. DDoS protection and rate limiting can also be added.

---
