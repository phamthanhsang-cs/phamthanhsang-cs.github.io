---
title: Endpoint Forensics - The Crime @CyberDefender
date: 2025-08-5 00:00:00 +0700
categories: [dfir, blue-teaming, cyber-defender]
tags: [digital-forensics, endpoint-forensics, android-forensic]
author: <author_id>
description: The Crime Lab @CyberDefender Walkthrough
# toc: false # Uncomment to disable the Table of Contents in the right panel
# comments: false # Uncomment to disable comments
image: /assets/images/preview/cyberdefenders_og.png
--- 


### Scenario

We're currently in the midst of a murder investigation, and we've obtained the victim's phone as a key piece of evidence. After conducting interviews with witnesses and those in the victim's inner circle, your objective is to meticulously analyze the information we've gathered and diligently trace the evidence to piece together the sequence of events leading up to the incident.

### Collected Data
- Android data ~ 700 MB in size

### Tools
- [ALEAPP](https://github.com/abrignoni/ALEAPP)
- [Everything](https://www.voidtools.com/downloads/)

### Questions

#### Q1: Based on the accounts of the witnesses and individuals close to the victim, it has become clear that the victim was interested in trading. This has led him to invest all of his money and acquire debt. Can you identify the `SHA256` of the trading application the victim primarily used on his phone?

```shell
git clone https://github.com/abrignoni/ALEAPP.git

cd ALEAPP

py -m pip install -r requirements.txt

python aleappGUI.py
```

First, ingest dataset into ALEAPP,  open `index.html`, below are things you should expected after data parsed successfully.

![pic1](assets/images/cyberdefender/thecrime/pic1.png)

For this type of question, i would like to look around **INSTALLED APPS**, while i was searching for the interesthing, there was an **App Icon** indicated a crypto trade app called **Olymp Trade**. 

![pic2](assets/images/cyberdefender/thecrime/pic2.png)

The package name **com.ticno.olymptrade** is useful to track it's data located on this phone.

![pic3](assets/images/cyberdefender/thecrime/pic3.png)

The path contains information about app installation is `\data\app\com.ticno.olymptrade-lKDfBXc8qLNF9F2eXSyBwg==`

Within that folder, you could manage to find the main package installation `base.apk` of **Olymp Trade**

![pic4](assets/images/cyberdefender/thecrime/pic4.png)

```shell
PS C:\Users\Administrator\Desktop\CyberDefender\EndpointForensics\138-The-Crime\temp_extract_dir\data\app\com.ticno.olymptrade-lKDfBXc8qLNF9F2eXSyBwg==> Get-FileHash -Algorithm SHA256 .\base.apk

Algorithm       Hash                                                                   Path
---------       ----                                                                   ----
SHA256          4F168A772350F283A1C49E78C1548D7C2C6C05106D8B9FEB825FDC3466E9DF3C       C:\Users\Administrator\Deskto...
```

> ANSWER: `4F168A772350F283A1C49E78C1548D7C2C6C05106D8B9FEB825FDC3466E9DF3C`
{: .prompt-info }

#### Q2: According to the testimony of the victim's best friend, he said, `"While we were together, my friend got several calls he avoided. He said he owed the caller a lot of money but couldn't repay now"`. How much does the victim owe this person?

This information was easily found by discovering SMS messages recorded, i found there was a messaage from `+201172137258` to our victim, require the sum of **250,000 EGP** ~ **$5,162**, for his/her debt.

![pic5](assets/images/cyberdefender/thecrime/pic5.png)

> ANSWER: `250000`
{: .prompt-info }


#### Q3: What is the name of the person to whom the victim owes money?

Since we found the number of the owner which is `+201172137258`, this information could easily extracted by navigate to **Contacts** in ALEAPP output.

![pic6](assets/images/cyberdefender/thecrime/pic6.png)

> ANSWER: `Shady Wahab`
{: .prompt-info }

#### Q4: Based on the statement from the victim's family, they said that on `September 20, 2023`, he departed from his residence without informing anyone of his destination. Where was the victim located at that moment?

By navigated to **Recent Activity**, i managed to found the answer, it turned out the victim was use Google Map, to track his own geo-location.

![pic7](assets/images/cyberdefender/thecrime/pic7.png)

![pic8](assets/images/cyberdefender/thecrime/pic8.png)

> ANSWER: `The Nile Ritz-Carlton`
{: .prompt-info }

#### Q5: The detective continued his investigation by questioning the hotel lobby. She informed him that the victim had reserved the room for 10 days and had a flight scheduled thereafter. The investigator believes that the victim may have stored his ticket information on his phone. Look for where the victim intended to travel.

This question is very interesting, not because it's level of difficult, but it's "human-being" behavior question.

> If you were the victim, or you are a human, or you were you, to save the ticket information, **where would you like to save it** ? By take a picture ? Download a digital picture of your ticket to your phone ? or take a note in your phone about ticket information.

I started looking at `data\media`, since it stores information about picture, video, etc.

Really, really lucky, i found an image called `Plane Ticket.png` in `\Downloads` folder inside of it

![pic9](assets/images/cyberdefender/thecrime/pic9.png)

![pic10](assets/images/cyberdefender/thecrime/pic10.png)

> ANSWER: `Las Vegas`
{: .prompt-info }

#### Q6: After examining the victim's Discord conversations, we discovered he had arranged to meet a friend at a specific location. Can you determine where this meeting was supposed to occur?

This question was easily solved, thanks to ALEAPP since they had their own module for Discord chat.

![pic11](assets/images/cyberdefender/thecrime/pic11.png)

![pic12](assets/images/cyberdefender/thecrime/pic12.png)

> ANSWER: `The Mob Museum`
{: .prompt-info }
