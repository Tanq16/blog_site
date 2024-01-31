---
title: Introduction to Threat Modeling
date: 2020-10-14 12:00:00 +0500
categories: [Computers and Security]
tags: [security,threat-modeling]
---

## Threat Modeling

Threat modeling is best applied continuously throughout a software development project. Following is a four question framework that helps understand threat modeling &rarr; What are we working on? What can go wrong? What are we going to do about it? Did we do a good job? The following are the steps to be taken for threat modeling &rarr;

1. Assessment Scope &rarr; Identifying tangible assets, like databases of information or sensitive files, understanding the capabilities provided by the application and valuing them.
2. Identify Threat Agents and Possible Attacks &rarr; Characterization of the different groups of people who might be able to attack the application, both insiders and outsiders performing inadvertent mistakes or malicious attacks.
3. Understand Existing Countermeasures
4. Identify Exploitable Vulnerabilities
5. Prioritize Identified Risks &rarr; For each threat, estimate a number for likelihood and impact factors to determine an overall risk or severity level.
6. Identify Countermeasures to Reduce Threat

## Generic questions for a threat modeling exercise

1. What is the scope of infrastructure covered? (he number of devices and servers, the type of servers)
2. What type of system is it? Client-Server based or service based?
3. What are the assets? What data is handled? Who is the provider? Who is the customer?
4. What technologies are being used for the application/software?
5. What kind of data is stored and where? Is the data store encrypted?
6. Who has access to the data store? How can it be accessed?
7. What are the entry points to the system? (points of entry for the attacker like website, service at port xx, etc.)
8. What functions does the application perform? Is any function privileged?
9. Is there an authorization system in place for privileged functions?
10. How many sub systems make up the entire application?
11. How do those sub systems communicate? Is the communication secure (SSL/TLS)? Is the communication?
12. Do usual actions employ Failsafe default and Least privilege?
13. What kind of authentication system is in place (OAuth, MFA, etc.)? How is authentication maintained over time?
14. What kind of data is logged and monitored by the system and/or the sub systems?
15. Do the logs contain sensitive information? Are log files accessible based on authorization?
16. Is there some kind of backup in place? Is the backup secured? Where is the backup stored? How is data sent there (transit)?
17. Looking at STRIDE, what sort of attacks are possible based on gathered information? (Spoofing, Tampering, Repudiation, Information disclosure, Denial of Service, Elevation of privileges)
18. Do the sub systems store different kinds of data in different places? How does one compromised sub system affect the other?
19. What are the current countermeasures in place?

## Resources

1. [Threat Modeling](https://owasp.org/www-community/Threat_Modeling)
2. [CRV2 App Threat Modeling](https://owasp.org/www-community/CRV2_AppThreatModeling)
3. [What is Identity and Access Management](https://www.cloudflare.com/en-in/learning/access-management/what-is-identity-and-access-management/)
