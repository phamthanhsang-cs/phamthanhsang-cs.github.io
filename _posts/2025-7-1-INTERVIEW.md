---
title: Can You Hunt Something From Nothing?
date: 2025-07-15 00:00:00 +0700
categories: [siem, blue-teaming, soc]
tags: [splunk, threat-hunting, log-analysis]    
author: <author_id>   
description: A real-world story of threat hunting from a mystery dataset provided in a SOC analyst interview process.
# toc: false # Uncomment to disable the Table of Contents in the right panel
# comments: false # Uncomment to disable comments
# image: /assets/images/preview/
---

### A Story Behind

Back in August of 2024, while I was mindlessly scrolling through Facebook, I came across a post where someone was asking for help with an interview task. They had been given a dataset by a company as part of the interview process for a SOC Analyst position and were asked a simple yet powerful question:

> **"Can you spot anything suspicious in this dataset?"**

What made it even more interesting was that the candidate was given a few days to thoroughly analyze the data and submit their findings which is something you don’t often see in typical interviews.

Curious and intrigued by this challenge, I reached out to the person and was fortunate enough to receive a copy of the dataset for my own analysis.

### Inside the Dataset

> Before diving into the dataset, there are a few important points to clarify:
> 
> 1. I'm not sure whether the data is **real or synthetic**. It could be sample data generated specifically for the hiring process, or it might be extracted from a real enterprise network environment.
> 2. Since the original data files included a company name in their titles, I have **removed** it along with any potentially sensitive information to avoid unintentional disclosure.
{: .prompt-warning }

The dataset includes four `.csv` files:

- `CorpADServer.csv` – contains information related to Active Directory
- `CorpFirewall.csv` – contains information related to firewall logs
- `CorpSchedule.csv` – contains information about employee work schedules
- `CorpSysLogs.csv`  – contains system log data (Syslog)

![pic1](assets/images/interview-data/pic1.png)
_Let's take a closer look at each of these files_

#### CorpADServer – Active Directory Logs

![pic2](assets/images/interview-data/pic2.png)

The Active Directory logs contain the following fields:

- **Timestamp**: Unix timestamp indicating when the event occurred  
- **User**: Username from the Active Directory environment  
- **Message**: Details about user activity (e.g., logon/logoff status and endpoint name)

#### CorpFirewall – Firewall Logs

![pic3](assets/images/interview-data/pic3.png)

The firewall logs contain the following fields:

- **Timestamp**: Unix timestamp of the event  
- **src_ip**: Source IP address  
- **src_port**: Source port  
- **dest_ip**: Destination IP address  
- **dest_port**: Destination port  
- **bytes_in**: Amount of inbound traffic (in bytes)  
- **bytes_out**: Amount of outbound traffic (in bytes)

#### CorpSchedule – Employee Work Schedules

![pic4](assets/images/interview-data/pic4.png)

The work schedule file contains the following fields:

- **Name**: Full name of the employee  
- **UserID**: Matches the **User** field in AD logs  
- **In**: Clock-in time  
- **Out**: Clock-out time  
- **Remote Access**: Indicates whether the employee is working remotely

#### CorpSysLogs – System Logs (Syslog)

![pic5](assets/images/interview-data/pic5.png)

The system logs contain the following fields:

- **Timestamp**: Unix timestamp of the event  
- **User**: Matches the **User** field in AD logs  
- **Host Name**: Name of the endpoint  
- **Message**: Describes user activity on the endpoint

### Pre-analysis

As you can see, the dataset is quite limited, there are no alerts from any security solutions, no detailed information, and no operating system event IDs.

To be honest, when I first looked at the data, I had no idea how I could find anything interesting within these logs.

However, this type of challenge truly tests an analyst’s ability to apply deep knowledge of **user behavior analysis** - understanding what's normal versus what's abnormal. It also demands a strong **threat hunting mindset**, where you proactively look for subtle indicators of suspicious activity without relying on traditional alerts or obvious signs.

#### Tools 

Use whatever tools you're most comfortable with—there’s no single "correct" choice for analysis.

In my case, I already had a Splunk on-premise instance set up for log management, so I decided to use it. Additionally, I needed to extract meaningful patterns from the dataset, rely heavily on visualizations, and perform correlation across multiple logs. Splunk is a powerful tool that excels in these areas and fits the purpose well.

#### Ingesting & Parsing 

Following the steps outlined in the [**Pre-analysis**](https://phamthanhsang-cs.site/posts/INTERVIEW-DATA/#pre-analysis) section, let’s walk through the process step by step, starting from ingesting the logs to parsing the data, and enriching it to add more context. 

The goal here is to transform raw logs into meaningful and actionable information that can support effective analysis.

First is ingesting the logs into Splunk, for maintainence, handling data purpose and also an isolated enviroment instead of touching global config in Splunk, i'll create an Splunk app.

1. Create separated app for dataset: Apps -> Manage Apps -> Create App

![pic6](assets/images/interview-data/pic6.png)

2. 
