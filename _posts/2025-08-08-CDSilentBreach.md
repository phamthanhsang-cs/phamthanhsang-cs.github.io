---
title: Endpoint Forensics - Silent Breach @CyberDefender
date: 2025-09-09 00:00:00 +0700
categories: [dfir, blue-teaming, cyber-defender]
tags: [digital-forensics, endpoint-forensics]
author: <author_id>
description: XLMRat Lab @CyberDefender Walkthrough
# toc: false # Uncomment to disable the Table of Contents in the right panel
# comments: false # Uncomment to disable comments
image: /assets/images/preview/cyberdefenders_og.png
--- 

### Scenario 

The IMF is hit by a cyber attack compromising sensitive data. Luther sends Ethan to retrieve crucial information from a compromised server. Despite warnings, Ethan downloads the intel, which later becomes unreadable. To recover it, he creates a forensic image and asks Benji for help in decoding the files.

Resources:
- [Windows Mail Artifacts: Microsoft HxStore.hxd (email) Research](https://boncaldoforensics.wordpress.com/2018/12/09/microsoft-hxstore-hxd-email-research/)

### Data Collected 
- ethanPC.ad1 ~ 700 MB in size

### Tools 
- FTK Imager
- SQLite Viewer
- Strings (Sysinternal Suite)

### Questions 

#### Q1: What is the MD5 hash of the potentially malicious EXE file the user downloaded?

Very first thing, yes ofc, ingest case evidence into FTK Imager.

![pic1](assets/images/cyberdefender/silentbreach/pic1.png)



#### Q2: What is the URL from which the file was downloaded?

#### Q3: What application did the user use to download this file?

#### Q4: By examining Windows Mail artifacts, we found an email address mentioning three IP addresses of servers that are at risk or compromised. What are the IP addresses?

#### Q5: By examining the malicious executable, we found that it uses an obfuscated PowerShell script to decrypt specific files. What predefined password does the script use for encryption?

#### Q6: After identifying how the script works, decrypt the files and submit the secret string.



