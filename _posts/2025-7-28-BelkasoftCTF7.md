---
title: BelkaCTF No.7 - Stranger Dfings Walkthrough
date: 2025-07-28 00:00:00 +0700  
categories: [dfir, blue-teaming]  
tags: [digital-forensics, belkactf]  
author: <author_id>  
description: Quick walkthrough on CTF Challenge from one of the most famous vendors in Digital Forensic realm ! 
# toc: false # Uncomment to disable the Table of Contents in the right panel
# comments: false # Uncomment to disable comments
image: /assets/images/preview/belka7.jpg
---

### Introduction 
Hi fellows, i just finished Capture the Flag challenge from Belkasoft, one of the most famous vendor in this realm.

Not gonna lie, it was one of the toughest one i got so far, particularly in Digital Forensic side.

I got ranked 7th as Student Scoreboard and 18th in overall scoreboard, feel a bit defeated but i will take it and embrace myself next year. 

![pic1](assets/images/belka/belka7/pic1.png)

I want to write a quick recap of what i'd done so far in the competition, also, prepare for the previous Belkasoft CTFs details forensics, and write-ups.

### About the Challenge

#### Information 
Belka CTF is the 48 hours Solo Capture the Flags Challenge from Belkasoft, one of the most famous vendor in the field.

BelkaCTF #7: Stranger Dfings, starts on July 25th, 2025 at 3 PM CEST / 1 PM UTC / 9 AM EDT and ended at July 27th, 2025 at 3:00 PM CEST.

Above 1600 participant, 300 investigators solved at least 1 questions.

About scoring, **dynamic scoring** will be used: the more participants solve a challenge, the lower its point value becomes for everyone who solved it. There is no speed bonus, points are decreased for everyone, including those who already solved the challenge. The winner is the one who scored more points than others did. If two competitors have the same amount of points, the one who reached that amount earlier wins.

And there will be two separated winners, among students and another among professionals.

#### Data Set
There are 3 dataset in this challenge:
- Memory Dump from the suspected Windows machine
- Extracted data from suspected phone (Android)
- Network Packet Captured from suspected criminal

If you wanna give it a try, below is the link to download those dataset, and it's archived password:
- [https://belkasoft.com/belkactf7/info](https://belkasoft.com/belkactf7/info)

#### Tools 
You can use whatever you want to get the flags from dataset, Belkasoft said some tasks are easier with Belkasoft X, but not mandatory.

From my perspective, you could 100% finish the challenge without Belksoft X (6th and even 7th winners didn't use Belkasoft X, yeah ofc he's from Russian xD)

For this challenge, i've used:
- **Belkasoft X** for the main forensic tool
- **Volatility3** and **MemProcFS** for memory analysis
- **DB Broswer (SQLite)** for reading .db file 
- **Foremost** to extract evidences from .dump file
- **dnSpy** for malware analysis
- **Wireshark** for network packet captured analysis

### Walkthrough

>From Question No.1 to No.7, the only evidence we got was the memory dump from suspected Windows machine.
{: .prompt-tip }

First thing is create a Case and feed data source into Belkasoft X, our first evidence we got is a piece of memory dump from suspected machine. 

![pic2](assets/images/belka/belka7/pic2.png)

After feeding memory dump successfully into Belkasoft Case, you will get something like this.

![pic3](assets/images/belka/belka7/pic3.png)

#### Question No.1: What is the username and hostname of the imaged machine? Format: user@hostname

For memory forensics stub, i figured out how much i like the MemProcFS for this kind of challenge, really, you could surfing data in RAM in file structured, also, it insanely fast as well.

```shell
MemProcFS.exe -f C:\Users\Administrator\Desktop\BelkaCTF7\BelkaCTF_7_CASE250722_KTSOAERO\BelkaCTF_7_CASE250722_KTSOAERO.mem -forensic 1
```

The information about host machine is under `/sys/computername.txt`, about the user, it under `sys/users/users.txt`

- computername.txt: TSO-ATC-CT412                   
- users.txt`: 

```bash
   # Username                         SID                                                                              
-----------------------------------------                                                                              
0000 award                            S-1-5-21-3557345689-3593742580-873049707-1001                                      
```

> ANSWER: `award@TSO-ATC-CT412`  
{: .prompt-info }

#### Question No.2: There is something rogue running on this machine. What is the malware executable file name?

Go Artifacts -> Malware Finder, i see a suspicous process called `epxlorer.exe`, seems like trying to rouge the `explorer.exe`

![pic4](assets/images/belka/belka7/pic4.png)

Go back into MemProcFS `M:\forensic\ntfs\1\Users\award\Downloads`, i could also where the file came from.

![pic5](assets/images/belka/belka7/pic5.png)

> ANSWER: `epxlorer.exe`  
{: .prompt-info }

#### Question No.3: What is the MD5 hash of the malware running on the machine?

This question is pretty tricky one, if you analysis the memory by Volatility3, dump the malware and calculate it's MD5.

I got the right answer by calculated in MemProcFS. Go `M:\name\epxlorer.exe-9920\files\modules` and caculate the rouge process inside it.

```shell
PS M:\name\epxlorer.exe-9920\files\modules> Get-FileHash -Algorithm MD5 .\epxlorer.exe

Algorithm       Hash                                                                   Path
---------       ----                                                                   ----
MD5             75EA94B54420C39DCD3D8CE574BA9D34                                       M:\name\epxlorer.exe-9920\files\modules\epxlorer.exe
```
> ANSWER: `75EA94B54420C39DCD3D8CE574BA9D34`  
{: .prompt-info }

#### Question No.4: What was the email address of the person spreading the malware?

From Belkasoft X Artifacts, go to Overview -> Mails, there was a Phishing email, came from "Information Technology Department" `<ktso.sec@mailforce.net>` to the victim `alex.ward67@posteo.us` (owner of suspected machine)

![pic6](/assets/images/belka/belka7/pic6.png)
_lmao Emergency Program for Xtinguishing Local Operating-system Related Error Reactions xDDDD_

> ANSWER: `ktso.sec@mailforce.net`  
{: .prompt-info }

#### Question No.5: What is the intruder-hosted Telegram API server used by the malware? Format: https://hostname.com/

Since the malware executable was found, plus, i had a sense of .net based malware.  

Extracted the `.exe` and inspect it using dnSpy, i successfully decompiled the malware, and there were a hardcoded url inside.

![pic7](assets/images/belka/belka7/pic7.png)

> ANSWER: `https://api-telegram-org.ctf.do/`
{: .prompt-info }

#### Question No.6: What is the Telegram Bot API key hard-coded into the malware executable? Format: 123456789:ABcdEFg09876qwERtyUIop54321hIJkl

Since there was no API key in decompiled `.exe`, i inspect the memdump of `.exe` located at `M:\name\epxlorer.exe-9920\minidump` during it execution, and found the api key.

![pic8](assets/images/belka/belka7/pic8.png)

>ANSWER: `8169267144:AAFwkhmXh71SMVcvFLantjTlwlRbT9HdJdU`
{: .prompt-info }

#### Question No.7: What is the phone number of the malware operator? Intl. phone format: +1234567890123

Since we collected `admin_telegram_id = "7474460026"`, and also it's API Key, i could invoke some APIs request to the Telegram Bot.

![pic9](assets/images/belka/belka7/pic9.png)

Search the user name on Telegram, i found the flag which is +995568988280

![pic10](assets/images/belka/belka7/pic10.png)

>ANSWER: `+995568988280`
{: .prompt-info }

---

> From next question, we got another piece of evidence which extracted data from suspected Android phone. 
{: .prompt-tip }

Now feed Belkasoft X with new data source we just collected from the challenge. 

Below is our new artifacts, collected from suspected mobile device.

![pic11](assets/images/belka/belka7/pic11.png)
_More evidences, more fun, right?_

*Beside Belkasoft X, there's a extremly strong Android forensic tool - ALEAPP, in case you wanna use opensource tools to solve the challenge*

#### Question No.8: What is the Google account this phone is set up with? Format: username@gmail.com

From Artifacts -> Overview, go to System files -> Android accounts, we could found the gmail account relate to this phone.

![pic12](assets/images/belka/belka7/pic12.png)

>ANSWER: `goggleslover93@gmail.com`
{: .prompt-info }

#### Question No.9: Where does the phone owner work and what’s his position? For example, Senior grill operator at Wendy’s

This flag was inside the mailbox, one of the conversations, related to his next upcoming article. From Artifact -> Structure, go to Mails -> EML

![pic13](assets/images/belka/belka7/pic13.png)

>ANSWER: `Investigative Journalist at Peach State Ledger`
{: .prompt-info }

#### Question No.10: What are the last 4 digits of the suspect’s bank card, and the contact phone number of his bank? For example: *1337, +1234567890123

This is the tricky one i'd encountered, first is to detect suspect's bank card, i found it in several emails, especially relate to some services require payment (Spotify, Bolt, McDonald, etc)

Go to Artifact -> Overview -> Email 

![pic14](assets/images/belka/belka7/pic14.png)

The bank name, i found it by surfing phone's browser history and saw severals result relate to Liberty Bank (Artifacts -> Structure -> Browsers -> Google -> URLs). 

![pic15](assets/images/belka/belka7/pic15.png)

Go to the bank website, i found the contact number which is `+995 32 2 55 55 00`

![pic16](assets/images/belka/belka7/pic16.png)

>ANSWER: `*7076, +995322555500`
{: .prompt-info }

#### Question No.11: Which dependency-causing substance is mentioned in the chat logs on the phone? Format: common name, e.g. ketamine

This question is tough, also, time comsuming, but i managed to found the flags, BY TRANSLATED EVERY JUNKY CHAT IN THIS PHONE.

Actually, i got the idea from the vendor, behind these questions, since they just released new AI-feature on their product called BelkaGPT.

The evidence i found was in message chat, after our victim got abducted by aliens. 

![pic17](assets/images/belka/belka7/pic17.png)

**C8H10N4O2** is **Caffeine**, actually i failed some questions due to did not translate conversation entirely.

> ANSWER: `Caffeine`
{: .prompt-info }

#### Question No.12: Where the two missing people—an activist and a teenager—can be retrieved? Format: lat,lon lat,lon — or just click the map

**I DIDN'T SOLVE THIS QUESTION**

Turns out, it was pretty...tricky to find the answer. In those chat logs, were 2 chat messages contain geolocation without any context. 

![pic20](assets/images/belka/belka7/pic20.png)
_The first one_


![pic21](assets/images/belka/belka7/pic21.png)
_The second one_

And those two coordinates was the answer for this question.

> ANSWER: 41.6990, 44.7572 41.7694, 44.8639 
{: .prompt-info }

---

> From this question, we got another piece of evidence which the traffic captured by tech squad at suspected home.
{: .prompt-tip}

#### Question No.13: What is the password of the suspect’s home Wi-Fi network?

While inspecting sniffed traffic, we definitely see the SSID="PINEAPPLENET" stood out frequently.

![pic22](assets/images/belka/belka7/pic22.png)

Back to Belkasoft X, i was able to figure out the suspect's home Wi-Fi network on Artifacts -> Overview -> Wireless configurations.

![pic23](assets/images/belka/belka7/pic23.png)

> ANSWER: `G0P1n3APL!`
{: .prompt-info }

#### Question No.14: What does the suspect call himself since recently? Format: new nickname, e.g. X Æ A-12

This question were pretty easy, if you have some knowledge relate to packet analysis, and also, the tool Wireshark itself. 

Our very first task is decrypt the sniffed traffics, first go to Wireshark -> Edit -> Preferences.

From Preferences Tab, go to Protocol and find IEEE 802.11 since it's Wireless LAN standard, you will able to decrypt entire traffic by config decryption key that we just found out from previous question.

![pic24](assets/images/belka/belka7/pic24.png)


And by inspecting sniffed traffic, i found a lot of images via SMB traffics

![pic25](assets/images/belka/belka7/pic25.png)

Export those images, i was able to find the answer

![pic26](assets/images/belka/belka7/pic26.png)

![pic27](assets/images/belka/belka7/pic27.png)

> ANSWER: `Y Æ B-12`
{: .prompt-info }

#### Question No.15: Figure out the moment when the suspect’s body started acting weird as precisely as you can. Provide timestamp with seconds in a common format, e.g. 2025-05-15 15:05:05 UTC

From Belkasoft Artifact, to to Overview -> Heart rate, this information come from `com.xiaomi.hm.health\databases` which is contains information about user health information.

![pic28](assets/images/belka/belka7/pic28.png)

You will see exact timestamp, when user heart pulse went abnormal.

> ANSWER: 2025-07-17 12:53:37 UTC
{: .prompt-info}

#### Question No.16: At which event the journalist became puppeteered by aliens? Format: full name of the event, e.g. 2nd Symposium on Creative Writing

**I DIDN'T SOLVE THIS QUESTION**

Turns out, user was use Speedtest, at event where aliens abducted him.

![pic29](assets/images/belka/belka7/pic29.png)

Examined it's database, it was **TruthTack News Summit 2025**

![pic30](assets/images/belka/belka7/pic30.png)
_Pretty tricky, huh?_

>ANSWER: `TruthTack News Summit 2025`
{: .prompt-info}

#### Question No.17: What is the car the jour… alien drives? Format: make, model, year, e.g. Dodge Caliber (2008)

**I DIDN'T SOLVE THIS QUESTION**

I was pretty upset since i wasnt able to find the answer, turns out, it was easier than i thought.

Since i have a tool call Everything to find any find with "keyword", the answer was in "carservice", came from **com.google.android.projection.gearhead**

![pic31](assets/images/belka/belka7/pic31.png)

```xml
<?xml version='1.0' encoding='utf-8' standalone='yes' ?>
<map>
    <boolean name="frx_activation_logged" value="true" />
    <string name="car_saved_setting_night_mode">2</string>
    <int name="projected_presentation_config_max_checks" value="10" />
    <boolean name="toll_card_sensor_enabled" value="true" />
    <int name="touchpad_focus_navigation_history_max_age_ms" value="4000" />
    <boolean name="Mazda_CX-60_2020_car_ev_settings_enabled" value="true" />
    <int name="car_connection_count" value="0" />
    <string name="car_tos_data">2</string>
    <long name="disconnect_time" value="1752309649149" />
    <boolean name="car_backup_valid" value="false" />
    <string name="car_tos_main">1</string>
    <int name="touchpad_focus_navigation_history_max_size" value="30" />
    <long name="last_focus_change_time" value="1752308738230" />
    <string name="car_tos_safety">1</string>
    <int name="projected_presentation_config_check_delay" value="20" />
    <boolean name="has_video_focus" value="true" />
</map>

```

>ANSWER: `Mazda CX-60 2020`
{: .prompt-info }

#### Question No.18: Mine the ATC software logs for serial numbers of the radars that spotted a UFO flying by. For example: L123456, M34567890, K234567

I was able to found the answer for this question, by extracted ATC process and inspect it .db file.

But the interesting is, when i used MemProcFS, there was no information relate to the Radar that detected the UFO, but Vol3 was helped me to do that.

![pic32](assets/images/belka/belka7/pic32.png)

From TABLE object, we found there was a call sign UFO with air_craft id of 36.

![pic33](assets/images/belka/belka7/pic33.png)

Filter TABLE detect by air_craft id of 36, i found several detections come from radar_id 1, 2, 3 and 4.

![pic34](assets/images/belka/belka7/pic34.png)

And the final answer was in radar_serials.

> ANSWER: `RDAD425319469B, RDA665200514A, RNN770196044B, RDA714203872B`
{ : .prompt-info}  

#### Question No.19: Pull the full UFO label reported by the radars, as it was shown on the ATC software display Format: UFO137-XXXX-XXXX-X

This question is pretty interesting, since they gave us a little hint:

```text
I dialed the ATC tower, feeling a little ridiculous even before the dispatcher picked up.

—Air Traffic Control, this better be important.

—Hey, it’s the forensics guy again. Did anything... weird happen last night on your terminal? Something off?

Silence on the other end.
—You know what... funny you should ask. The radar software went all glitchy, picking up some...
unidentified something. Thought it was acting up again.

—Did you log it?

—Nah. Just took a quick screenshot and left it hanging unsaved in Paint or whatever.

—Seriously? You left crucial radar evidence in Paint?

He scoffed.
—Don’t lecture me, genius. Just do your magic voodoo thing and pull it.
```

I was able to solve this question, first by extract memdump from mspaint.exe, and then carve all data (relate to images) by `foremost`

```shell
foremost -t jpeg,png,jpg -i pid.8508.dmp -o /image
```

![pic35](assets/images/belka/belka7/pic35.png)

> ANSWER: `UFO137-1299-N21C-2`
{ : .prompt-info} 

#### Question No.20: What identification number is painted on board of the aliens’ spacecraft? Please help yourself to the NSA spy satellite imagery archive: spysatarchive.nsa.fyi/

**I DIDN'T SOLVE THIS QUESTION**

#### Question No.21: Connect to the aliens’ mothership VPN and start exploring. Where is the aliens’ craft parked right now? Format: lat,lon — or just click the map

**I DIDN'T SOLVE THIS QUESTION**

#### Question No.22: To save the Earth we need to instead point the beam at something useless... How about Pluto? Tell mothership to shoot Pluto to win the CTF.

**I DIDN'T SOLVE THIS QUESTION**

