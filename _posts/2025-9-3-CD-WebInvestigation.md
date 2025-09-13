---
title: Network Forensics - Web Investigation @CyberDefender
date: 2025-08-30 00:00:00 +0700
categories: [dfir, blue-teaming, cyber-defender]
tags: [digital-forensics, network-forensics]
author: <author_id>
description: Web Investigation Lab @CyberDefender Walkthrough
# toc: false # Uncomment to disable the Table of Contents in the right panel
# comments: false # Uncomment to disable comments
image: /assets/images/preview/cyberdefenders_og.png
--- 

### Scenario 
You are a cybersecurity analyst working in the Security Operations Center (SOC) of BookWorld, an expansive online bookstore renowned for its vast selection of literature. BookWorld prides itself on providing a seamless and secure shopping experience for book enthusiasts around the globe. Recently, you've been tasked with reinforcing the company's cybersecurity posture, monitoring network traffic, and ensuring that the digital environment remains safe from threats.

Late one evening, an automated alert is triggered by an unusual spike in database queries and server resource usage, indicating potential malicious activity. This anomaly raises concerns about the integrity of BookWorld's customer data and internal systems, prompting an immediate and thorough investigation.

As the lead analyst in this case, you are required to analyze the network traffic to uncover the nature of the suspicious activity. Your objectives include identifying the attack vector, assessing the scope of any potential data breach, and determining if the attacker gained further access to BookWorld's internal systems.

### Data Collected 
- WebInvestigation.pcap ~ 29 MB in size

### Tools 
- Wireshark
- NetworkMiner

### Questions 

After ingested case data into Wireshark, you should've something like below

![pic1](assets/images/cyberdefender/webinvestigation/pic1.png)

#### Q1: By knowing the attacker's IP, we can analyze all logs and actions related to that IP and determine the extent of the attack, the duration of the attack, and the techniques used. Can you provide the attacker's IP?

> Before performing any in-depth network analysis, i suggest taking a bird's eye view of the case metadata. Wireshark is a powerful tool to examine not only deep packet inspection but also `.pcap` metadata and statistics.
{: .prompt-tip }

![pic2](assets/images/cyberdefender/webinvestigation/pic2.png)

There's actually a ton of information that could be helpful in a real incident from a network forensic perspective, such as the incident-related timestamp, or even in some cases, information like OS, network interfaces, total packets, bytes, etc.

The next step, and in my opinion the most important step before analyzing any packet payload, is to examine the packet capture file statistics. 

![pic3](assets/images/cyberdefender/webinvestigation/pic3.png)

The first information i would like to look into is **Protocol Hierarchy**, this information helps us answer several questions related to normal or abnormal protocols within your environment (e.g: if your company does not use any SMB-related services, is it worth taking a look into captured packets that contain SMB traffic? You answer it yourself). In this case, we can observe that 99% of traffic is HTTP and a few file types (PNG, GIF, etc), so it might be easy to assume we are investigating an incident related to a web server.  

The next thing we want to examine is the packet capture conversation: how many hosts were talking or connecting to this web server in our case data.  

![pic4](assets/images/cyberdefender/webinvestigation/pic4.png)

By looking into the above conversation, we'll be able to extract receivers and senders in our case. For this particular example:
- `73.124.22.98` - Victim machine, which is the web server 
- `111.224.250.131` - Sender #1, took the most traffic in our case 
- `170.40.150.126` - Sender #2 

While Wireshark is a powerful tool for packet analysis, in this case, we also want to utilize one more tool - [Network Miner](https://www.netresec.com/?page=NetworkMiner).  

> The idea behind using such a tool is that Network Miner is really good at categorizing traffic into separated useful sections such as **Detailed Host Information**, **Credentials extracted from unencrypted traffic**, etc.
{: .prompt-tip }

![pic5](assets/images/cyberdefender/webinvestigation/pic5.png)

Inspecting host details on both senders, we can determine which host is responsible for malicious activities on the web server.

![pic6](assets/images/cyberdefender/webinvestigation/pic6.png)  
_`170.40.150.126` host information, looks normal at first_

![pic7](assets/images/cyberdefender/webinvestigation/pic7.png)  
_From the user-agent string, this host has a bunch of red team/hacking tools_

Right after examining each sender's detailed information, we extracted a bunch of hacking tools that had been used by the adversary to try exploiting the web server:
- [sqlmap](https://sqlmap.org/) -> SQL mapping, scanner, commonly use for SQL injection
- [gobuster](https://github.com/OJ/gobuster) -> Directory scanner, brute forcing
- Red Team framework, in this case, [Cobalt Strike](https://www.cobaltstrike.com/) or [Meterpreter](https://docs.metasploit.com/docs/using-metasploit/advanced/meterpreter/) -> Adversary Simulation Frameworks, adversary could utilize such legal frameworks for data exfiltration

> ANSWER: `111.124.250.131`
{: .prompt-info }


#### Q2: If the geographical origin of an IP address is known to be from a region that has no business or expected traffic with our network, this can be an indicator of a targeted attack. Can you determine the origin city of the attacker?

Since suspicious IP Address was identified, we could extract more information relate to this IP Address via OSINT (IP Lookup, Whois, AbuseIPDB, etc)

Below is information extracted from [AbuseIPDB](https://www.abuseipdb.com/check/111.224.250.131), sender IP Address was from China, Shijiazhuang, Hebei.

![pic8](assets/images/cyberdefender/webinvestigation/pic8.png)

Worth notice in above picture is, there's no report to this IP Address so far, in many cases, we often overlook such as IP Address if rely on OSINT totally w/out any internal knowledges.

> ANSWER: `Shijiazhuang`
{: .prompt-info }

#### Q3: Identifying the exploited script allows security teams to understand exactly which vulnerability was used in the attack. This knowledge is critical for finding the appropriate patch or workaround to close the security gap and prevent future exploitation. Can you provide the vulnerable PHP script name?

> ANSWER: `search.php`
{: .prompt-info }

#### Q4: Establishing the timeline of an attack, starting from the initial exploitation attempt, what is the complete request URI of the first SQLi attempt by the attacker?

> ANSWER: `/search.php?search=book and 1=1; -- -`
{: .prompt-info }

#### Q5: Can you provide the complete request URI that was used to read the web server's available databases?


> ANSWER: `/search.php?search=book' UNION ALL SELECT NULL,CONCAT(0x7178766271,JSON_ARRAYAGG(CONCAT_WS(0x7a76676a636b,schema_name)),0x7176706a71) FROM INFORMATION_SCHEMA.SCHEMATA-- -`
{: .prompt-info }
#### Q6: Assessing the impact of the breach and data access is crucial, including the potential harm to the organization's reputation. What's the table name containing the website users data?

> ANSWER: `customers`
{: .prompt-info }

#### Q7: The website directories hidden from the public could serve as an unauthorized access point or contain sensitive functionalities not intended for public access. Can you provide the name of the directory discovered by the attacker?

> ANSWER: `/admin/`
{: .prompt-info } 

#### Q8: Knowing which credentials were used allows us to determine the extent of account compromise. What are the credentials used by the attacker for logging in?

> ANSWER: `admin:admin123!`
{: .prompt-info }

#### Q9: We need to determine if the attacker gained further access or control of our web server. What's the name of the malicious script uploaded by the attacker?

> ANSWER: `NVri2vhp.php`
{: .prompt-info }

