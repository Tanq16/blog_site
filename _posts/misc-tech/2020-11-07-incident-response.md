---
title: Introduction to Incident Response
date: 2020-11-07 12:00:00 +0500
categories: [Computers and Security]
tags: [security,incident-response]
---

## BHIS IR Card game.

The following sections are taken from a card game made by Black Hills Information Security (BHIS).

### Injects

1. Management has approved the release of a new procedure → Once in a while, the C-suite approves some procedures, etc. which can be useful.
2. The main person who runs the IR process can be involved in explanatory meetings that can cause the other members to feel pressured.
3. An intern can reek damage on systems that are being reviewed by IR personnel.
4. Lead handlers can be on emergency leaves which can cause issues in handling situations.
5. Analysts can be trained in special respects which can be an asset to the teams.
6. Test situations can be set up, for example → Management can hire a Red team for assessment, which can also be a test on the IR.
7. Deploying honeypots can be effective to handle some sort of response.

### Initial Compromise

1. Credential Stuffing → The attackers take advantage of third party breaches to identify and use IDs and passwords against your organization. Detection → Server Analysis, UEBA
2. Exploitable External Service → An external service could have a misconfiguration or a publicly available exploit that the attackers can take advantage of, to attack and pivot to internal resources. Detection → Firewall Log Review, Server Analysis
3. Exploited BYOD (Bring Your Own Device) → This can be used as an entry point to compromising organizational networks. Detection → Firewall Log Review, Netflow, Zeek/Bro, RITA
4. Social Engineering → Tricking users to download malware. Detection → Endpoint Security Protection Analysis, User Awareness Training
5. Trusted Relationship → A trusted third party who has access to the network can be an entry point if compromised. Detection → SIEM Log analysis, UEBA
6. Password Sprays → Spraying commonly used passwords in the network. Detection → SIEM Log analysis, UEBA, Firewall Log Review
7. Insider Threat → An internal disgruntled user exfiltrates information from the organizational network. Detection → UEBA, Working with HR, DLP is a false hope
8. External Cloud Access → The attackers can use cloud resources to gain access. Detection → SIEM Log analysis
9. Web Server Compromise → The attackers take over an external web server and use it to pivot to the organizational network. Detection → Server Analysis, SIEM Log analysis, Netflow, Zeek/Bro, RITA
10. Phish → The attackers send a malicious email targeting users because they are easy to attack. Detection → Firewall Log review, Endpoint Security Protection Analysis

### Pivot and Escalate

1. Local Privilege Escalation → Attackers use a vulnerability in the local software to gain administrative access. Detection → Endpoint Analysis, Endpoint Security Protection Analysis
2. New Service Creation → Attackers create and load their malware using a service with system/root privileges, or create a new service. Detection → Endpoint Analysis, Endpoint Security Protection Analysis
3. Accessibility Features → The attackers hijack accessibility features like sticky keys and onscreen keyboard. Detection → Endpoint Analysis, Endpoint Security Protection Analysis
4. Credential Stuffing → Valid AD credentials have been discovered on open shares and files within the environment. Detection → SIEM Log analysis, UEBA, Internal Segmentation
5. Weaponizing AD → The attacker map trust relationships and user/group privileges in the AD network. Detection → SIEM Log analysis, UEBA, Internal Segmentation
6. Broadcast/Multicast Protocol Poisoning → LLMNR (Link Local Multicast Name Resolution) lets a host ask for name resolution from any system on the same network. The attackers perform the poisoning on the AD network. Detection → CredDefense Toolkit, UEBA, Firewall Log review
7. Kerberoasting → The attackers use a feature of SPNs (Service Principle Names) to extract and crack service passwords. Detection → SIEM Log analysis, UEBA, Honey Services, Internal Segmentation
8. Internal Password Spray → The attackers start a password spray against the rest of the organization from a compromised system. Detection → UEBA, SIEM Log analysis

### Persistence

1. Evil Firmware → Attackers update the firmware of the Network Cards, Video Cards and BIOS (or UEFI) with malicious ones. These are difficult to detect as well as to update. Detection → Endpoint Analysis, Endpoint Security Protection Analysis, Prayers to God (lol)
2. Logon Scripts → Attackers install a script that triggers when a user logs on. Detection → Endpoint Analysis, Endpoint Security Protection Analysis
3. Malicious Browser Plugins → Attackers install plugins in the browser, which can be used as part of C2 and persistence. The browser becomes the new endpoint. Detection → Endpoint Analysis, Endpoint Security Protection Analysis, Firewall Log review, Netflow, Zeek/Bro, RITA
4. Application Shimming → Attackers use the Application Compatibility Toolkit to trick applications into not seeing the ports, directories, files and services the attackers want to hide. Detection → Endpoint Analysis, Endpoint Security Protection Analysis
5. New User added → The attackers add a new user to local computers. Detection → Endpoint Analysis, Endpoint Security Protection Analysis
6. Malicious Driver → The attackers load a malicious driver into the OS. Detection → Endpoint Analysis, Endpoint Security Protection Analysis
7. DLL Attacks → The attackers hijack the order in which DLLs are loaded. This is usually done through insecure permissions. Detection → Endpoint Analysis, Endpoint Security Protection Analysis
8. Malicious Service or Malware → The attackers add a service that starts every time the system starts. Detection → Endpoint Analysis, Endpoint Security Protection Analysis

### C2 and Exfil

1. Domain Fronting as C2 → The attackers use Domain Fronting to bounce their traffic off of legitimate systems. Detection → Netflow, Zeek/Bro, RITA
2. Gmail, Tumblr, Salesforce, Twitter as C2 → The attackers route traffic through third party services, many of which are ignored completely by many security tools. Detection → Netflow, Zeek/Bro, RITA
3. Windows BITS (Background Intelligent Transfer Service) → The attackers use BITS, a protocol that is often ignored. Detection → Netflow, Zeek/Bro, RITA
4. DNS as a C2 channel. Detection → Netflow, Zeek/Bro, RITA
5. HTTPS as Exfil → Many malwares use this. Example → Meterpreter has used this since long. This can be used in conjunction with other stego techniques. Detection → Netflow, Zeek/Bro, RITA
6. HTTP as Exfil → This is usually used in conjunction with some form of stego like → VSAgent uses base64 encoded `__VIEWSTATE` as an exfil field. Detection → Netflow, Zeek/Bro, RITA

### Procedures

1. Crisis Management → The legal and management teams have procedures for effectively and ethically notifying impacted victims of compromises. A good notification strategy will help deal with a political fallout.
2. Isolation → The network team must isolate infected systems to prevent further harm after an incident.
3. Endpoint Analysis → Defenders use IR cheat sheets like SANS to detect attacks on workstations. This requires interaction with the members sitting on Help Desk.
4. UEBA (User and Entity Behavior Analytics) → It looks for multiple concurrent logins, impossible logins based on geography, unusual file access, password sprays, etc.
5. Endpoint Security Protection Analysis → AV on endpoints must always generate logs which should be monitored and not forgotten.
6. Internal Segmentation → Internal networks must be segmented i.e., treating segments as hostile. Host based firewalls help in this case.
7. Netflow, Zeek/Bro, RITA (Real Intelligence Threat Analysis) → Network traffic must be captured, parsed and reviewed following a documented process. Just running tools is not enough.
8. Firewall Log Review → Emulate attack scenarios and verify the working of procedures in place. Logs from firewalls must be analyzed and understood.
9. SIEM (Security Information and Event Management) → Regular attack scenario emulations can be used to check if they can be detected and logged.
10. Server Analysis → The ability to baseline a system and verify that it is operating in a normal state.

---

## Incident Response

### PREPARATION

Preparation is the key to effective [**incident response**](https://digitalguardian.com/blog/what-incident-response). Even the best incident response team cannot effectively address an incident without predetermined guidelines. A strong plan must be in place to support your team. In order to successfully address security events, these features should be included in an incident response plan:

- **Develop and Document IR Policies:** Establish policies, procedures, and agreements for incident response management.
- **Define Communication Guidelines:** Create communication standards and guidelines to enable seamless communication during and after an incident.
- **Incorporate Threat Intelligence Feeds:** Perform ongoing collection, analysis, and synchronization of your threat intelligence feeds.
- **Conduct Cyber Hunting Exercises:** Conduct operational threat hunting exercises to find incidents occurring within your environment. This allows for more proactive incident response.
- **Assess Your Threat Detection Capability:** Assess your current threat detection capability and update risk assessment and improvement programs.

The following resources may help you develop a plan that meets your company’s requirements:

- [NIST Guide: Guide to Test, Training, and Exercise Programs for IT Plans and Capabilities](http://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-84.pdf)
- [SANS Guide: SANS Institute InfoSec Reading Room, Incident Handling, Annual Testing and Training](https://www.sans.org/reading-room/whitepapers/incident/incident-handling-annual-testing-training-34565)

### DETECTION AND REPORTING

The focus of this phase is to monitor security events in order to detect, alert, and report on potential security incidents.

- **Monitor:** Monitor security events in your environment using firewalls, intrusion prevention systems, and data loss prevention.
- **Detect:** Detect potential security incidents by correlating alerts within a SIEM solution.
- **Alert:** Analysts create an incident ticket, document initial findings, and assign an initial [incident classification](https://digitalguardian.com/blog/creating-incident-response-classification-framework).
- **Report:** Your reporting process should include accommodation for regulatory reporting escalations.

### TRIAGE AND ANALYSIS

The bulk of the effort in properly scoping and understanding the security incident takes place during this step. Resources should be utilized to collect data from tools and systems for further analysis and to identify indicators of compromise. Individuals should have in-depth skills and a detailed understanding of live system responses, digital forensics, memory analysis, and malware analysis. As evidence is collected, analysts should focus on three primary areas:

- Endpoint Analysis
   - Determine what tracks may have been left behind by the threat actor.
   - Gather the artifacts needed to build a timeline of activities.
   - Analyze a bit-for-bit copy of systems from a forensic perspective and capture RAM to parse through and identify key artifacts to determine what occurred on a device.
- Binary Analysis
   - Investigate malicious binaries or tools leveraged by the attacker and document the functionalities of those programs. This analysis is performed in two ways.
      1. Behavioral Analysis: Execute the malicious program in a VM to monitor its behavior
      2. Static Analysis: Reverse engineer the malicious program to scope out the entire functionality.
- Enterprise Hunting
   - Analyze existing systems and event log technologies to determine the scope of compromise.
   - Document all compromised accounts, machines, etc. so that effective containment and neutralization can be performed.

### CONTAINMENT AND NEUTRALIZATION

This is one of the most critical stages of incident response. The strategy for containment and neutralization is based on the intelligence and indicators of compromise gathered during the analysis phase. After the system is restored and security is verified, normal operations can resume.

- **Coordinated Shutdown:** Once you have identified all systems within the environment that have been compromised by a threat actor, perform a coordinated shutdown of these devices. A notification must be sent to all IR team members to ensure proper timing.
- **Wipe and Rebuild:** Wipe the infected devices and rebuild the operating system from the ground up. Change passwords of all compromised accounts.
- **Threat Mitigation Requests:** If you have identified domains or IP addresses that are known to be leveraged by threat actors for command and control, issue threat mitigation requests to block the communication from all egress channels connected to these domains.

### POST-INCIDENT ACTIVITY

There is more work to be done after the incident is resolved. Be sure to properly document any information that can be used to prevent similar occurrences from happening again in the future.

- **Complete an Incident Report:** Documenting the incident will help to improve the incident response plan and augment additional security measures to avoid such security incidents in the future.
- **Monitor Post-Incident:** Closely monitor for activities post-incident since threat actors will re-appear again. We recommend a security log hawk analyzing SIEM data for any signs of indicators tripping that may have been associated with the prior incident.
- **Update Threat Intelligence:** Update the organization’s threat intelligence feeds.
- **Identify preventative measures:** Create new security initiatives to prevent future incidents.
- **Gain Cross-Functional Buy-In:** Coordinating across the organization is critical to the proper implementation of new security initiatives.

---

## Resources

1. [The Five Steps of Incident Response](https://digitalguardian.com/blog/five-steps-incident-response)
2. [Black Hills Information Security](https://www.blackhillsinfosec.com/projects/backdoorsandbreaches/)

## More Resources

1. [Incident Response Playbooks Gallery](https://www.incidentresponse.com/playbooks/)
2. [Incident Response SANS: The 6 Steps in Depth](https://www.cynet.com/incident-response/incident-response-sans-the-6-steps-in-depth/)

