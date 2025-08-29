---
title: BelkaCTF No.1- BelkaDay Walkthrough 
date: 2025-09-10 00:00:00 +0700 
categories: [dfir, blue-teaming]  
tags: [digital-forensics, belkactf]  
author: <author_id>  
description: Quick walkthrough on BelkaCTF Challenge#1 - BelkaDay 
# toc: false # Uncomment to disable the Table of Contents in the right panel
# comments: false # Uncomment to disable comments
image: assets/images/preview/belkasoftctf1.jpg
---

### The plot

You were contacted by a company preparing their new product launch: an AI-based recommendation system that respects target privacy. Just before the date, the source code and technical documents ended up in their competitor's hands. The company suspects a recently hired developer and obtained a copy of his corporate laptop HDD. You are going to analyze the image and support the suspicion with evidence extracted from the laptop...


### Challenges

#### Question No.1: First let's identify the subject. What is the full name of the laptop owner? Format: First name Last name

After ingest case data into Belkasoft X, you will have something look like this.

![pic1](assets/images/belka/belka1/pic1.png)

Go to Artifacts -> Overview -> Windows -> User name and Sid list, 
