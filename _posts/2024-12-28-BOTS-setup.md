---
title: Setup your own Boss of the SOC 
date: 2024-12-25 10:20:00 + 0700
categories: [Homelab, SIEM]
tags: [splunk, linux]    
author: <author_id>   
description: Spin up your own blue-team CTF Challenge with Boss of the SOC by Splunk
# comments: false 
image: /assets/images/preview/botssetuppreview.png
mermaid: true
---
## Introduction
I’ve always enjoyed learning about cybersecurity especially on blue side through hands-on challenges, and CTF - Boss of the SOC (BOTS) by Splunk are perfect for that. They give you the chance to experience real-world scenarios in a way that's engaging and practical.

When I decided to set up BOTS and a CTF dashboard for myself, I was surprised at how little information was out there. Most of the guides I found were either focused on pre-hosted setups on online platforms, or only stop at install the Boss of the SOC data set. 

That’s why I decided to write this blog. I wanted to share my experience, step by step, so anyone interested in doing the same thing can have a clear starting point. If you’ve been wanting to self-host BOTS or your own CTF dashboard but didn’t know where to begin, I hope this helps make the process easier and more approachable.
<br>

## Let’s Dive In!

You can run your Splunk on any OS: Windows, Linux or MacOS. I decided to go with Linux and set it up on a [Proxmox](https://www.proxmox.com/en/) virtual machine. That said, you can host it on whatever platform works best for you, like VMware Workstation, VirtualBox, Hyper-V, or even any Cloud hosting.

![LinuxBox](/assets/images/bots-setup/neofetch-linuxboxinformation.png){: w="550" h="50" }{: .right }

- Linux Distro: Debian 12
- vCPU: 8 cores
- 16 GB of RAM
- 50 GB of SSD Storage
- `root` or `sudo` user
<br>
<br>
<br>
<br>
<br>

Make sure your Linux box is fully upgraded.
```bash
apt update && apt upgrade -y
```

Create `botsv1` directory, i like to put my `botsv1` data into that directory for re-use purposes, and Splunk in `/otp/` directory.
```bash
mkdir /opt/botsv1 && cd /opt/
```

Download Splunk Enterprise and unzip your download, i choose Splunk version 8.2.9 for my BOTSv1.
```bash
wget -O splunk-8.2.9-4a20fb65aa78-Linux-x86_64.tgz "https://download.splunk.com/products/splunk/releases/8.2.9/linux/splunk-8.2.9-4a20fb65aa78-Linux-x86_64.tgz" && tar xvzf splunk-8.2.9-4a20fb65aa78-Linux-x86_64.tgz -C /opt
```

Setup Splunk path for easy navigation.
```bash
export SPLUNK_HOME=/opt/splunk
```

Enable Splunk boot start which is help your Splunk always up even when you reboot your linux box.
```bash
$SPLUNK_HOME/bin/splunk enable boot-start 
     ...
     Moving '/opt/splunk/share/splunk/search_mrsparkle/modules.new' to '/opt/splunk/share/splunk/search_mrsparkle/modules'.
     Init script installed at /etc/init.d/splunk.
     Init script is configured to run at boot.
```
Setup Splunk admin credential:
- Please enter an administrator username: mine is `sys-admin`
- Please enter a new password: (setup your own password)

Start your Splunk Enterprise.
```bash
systemctl enable splunk
systemctl restart splunk
systemctl status splunk
...
botsv1 splunk[1216]:         All installed files intact.
botsv1 splunk[1216]:         Done
botsv1 splunk[1216]: All preliminary checks passed.
botsv1 splunk[1216]: Starting splunk server daemon (splunkd)...
botsv1 splunk[1216]: Done
botsv1 splunk[1216]: Waiting for web server at http://127.0.0.1:8000 to be available....... Done
botsv1 splunk[1216]: If you get stuck, we're here to help.
botsv1 splunk[1216]: Look for answers here: http://docs.splunk.com
botsv1 splunk[1216]: The Splunk web interface is at http://botsv1:8000
botsv1 systemd[1]: Started splunk.service - LSB: Start splunk.
```

(Optional) You can confirm that your Splunk is up by using net-tools.
```bash
netstat -tlnpd
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      592/sshd: /usr/sbin 
tcp        0      0 0.0.0.0:8089            0.0.0.0:*               LISTEN      1287/splunkd        
tcp        0      0 0.0.0.0:8191            0.0.0.0:*               LISTEN      1375/mongod         
tcp        0      0 0.0.0.0:8000            0.0.0.0:*               LISTEN      1287/splunkd  <--- this one       
tcp        0      0 127.0.0.1:8065          0.0.0.0:*               LISTEN      1472/python3.7      
tcp6       0      0 :::22                   :::*                    LISTEN      592/sshd: /usr/sbin 
```
Go to browser, enter your Splunk instance IP Address with port 8000 and sign-in with your previous Splunk admin credential.
![firstimelogin](/assets/images/bots-setup/firstimelogin.png)
_First login GUI of Splunk by default_

Go back to your Linux machine, download [`botsv1`](https://github.com/splunk/botsv1) dataset. The version I downloaded contains full data (around 33 million events), including both attack and normal (noisy) data. This dataset will help enhance our Splunk search capabilities. 

*For context, in my previous company, , i tried sending Fortigate Firewall syslogs into SC4S, which then forwarded the logs to Splunk. After about 30 days, we had around 128 million event records, consuming about 33GB of storage. It was crazy!*

```bash
cd /opt/botsv1
wget https://s3.amazonaws.com/botsdataset/botsv1/splunk-pre-indexed/botsv1_data_set.tgz
```

Extract your dataset and put in to Splunk apps directory.
```bash
tar xvzf botsv1_data_set.tgz -C $SPLUNK_HOME/etc/apps
```

Install these Apps / Add-on, follow [here](https://docs.splunk.com/Documentation/AddOns/released/Overview/Singleserverinstall) if you don't know how to install Splunk App and Add-on.

*Note: some apps/add-on already archived or did not support anymore, but you still can download almost of those app without any issue.*

| App / Add-on                         | Link                                    |
| ------------------------------------ | --------------------------------------- |
| Fortinet Fortigate Add-on for Splunk | https://splunkbase.splunk.com/app/2846/ |
| Splunk App for Stream                | https://splunkbase.splunk.com/app/1809/ |
| Splunk Add-on for Microsoft Windows  | https://splunkbase.splunk.com/app/742/  |
| Splunk TA for Suricata               | https://splunkbase.splunk.com/app/2760/ |
| Splunk Add-On for Microsoft Sysmon   | https://splunkbase.splunk.com/app/1914/ |
| URL Toolbox                          | https://splunkbase.splunk.com/app/2734/ |

You will not have Splunk Add-on for Tenable	here since it archived already, but feel free to contact [me](https://t.me/sangpham0311) for it.

Now you are able to searching the data of BOTSv1 with this SPL Command:
```SQL
index=botsv1 earliest=0
```
![firstimesearch](/assets/images/bots-setup/firstsplsearch.png)
_We are now able to search BOTSv1 Data in Splunk_

But we don't stop here, let's setup our own Boss of the SOC CTF Scoreboard ! 

Firtly, install these Apps / Add-on. 

| App / Add-on                                | Link                                                                         |
| ------------------------------------------- | ---------------------------------------------------------------------------- |
| Splunk App for Lookup File Editing          | https://splunkbase.splunk.com/app/1724/                                      |
| Parallel Coordinates - Custom Visualization | Already archived, contact [me](https://t.me/sangpham0311) to get `.tgz` file |
| Simple Timeseries Custom Visualization      | Already archived, contact [me](https://t.me/sangpham0311) to get `.tgz` file |
| Timeline Custom Visualization               | Already archived, contact [me](https://t.me/sangpham0311) to get `.tgz` file |

Move to Splunk `apps` directory and install CTF Scoreboard and CTF Scoreboard admin app.
```bash
cd $SPLUNK_HOME/etc/apps
git clone https://github.com/splunk/SA-ctf_scoreboard
git clone https://github.com/splunk/SA-ctf_scoreboard_admin
```

Restart Splunk for prerequisites and the scoring apps recognition
```bash
systemctl restart splunk
```

Create `scoreboard` log directory, you will need this.
```bash
mkdir $SPLUNK_HOME/var/log/scoreboard
```

Create `svcaccount` which is CTF Answers service account.
```bash
$SPLUNK_HOME/bin/splunk add user svcaccount -password <password> -role ctf_answers_service -auth admin:<admin_password>
```

Config the custom controller in `SA-ctf_scoreboard` app.
```bash
cd $SPLUNK_HOME/etc/apps/SA-ctf_scoreboard/appserver/controllers
cp scoreboard_controller.config.example scoreboard_controller.config
nano scoreboard_controller.config
```
```shell
[ScoreboardController]
USER = svcaccount
PASS = <your svcaccount password>
VKEY = random string (10-20 characters)
```

Restart your Splunk instance, we are done with terminal and stuffs like that, let's go back to your Splunk Web GUI.

Now, let's setup BOTS admin user, you could use Administrator user by default, but it need the following roles:
- admin
- ctf_admin
- can_detele 

In my case, i'll create 2 users: one for BOTSv1 Admin and another is for us - competitors 

![BOTSv1 Admin gif video](/assets/images/bots-setup/addbotsadmin.gif)
_BOTS Admin Setup_

![BOTSv1 Competitor gif video](/assets/images/bots-setup/addcompetitoruser.gif)
_BOTS Competitor Setup_

Load sample data into Capture the Flag admin (Don't forget using `BOTS Admin` user).

![Load sample data](/assets/images/bots-setup/loadsampledata.gif)
_Example: load some types of sample data_

Load BOTS questions / answers / hints, you could send email to bots@splunk.com (BOTS Team) to get those contents or contact [me](https://t.me/sangpham0311) for it. 
 - Move to Capture the Flag Admin app --> Edit Questions / Answers / Hints --> Import --> Select `.csv` file

![Edit Question data](/assets/images/bots-setup/editquestioncsv.gif)
_Example: Load question .csv to BOTS questions_

After your data import finish, you could attest by navigate to Capture the Flag Admin --> View --> Q&A

Now set the start / end time of the questions by navigate to Capture the Flag Admin --> Edit --> Time Setup and then `Submit Changes` since we are not have any pressure about timely so we could left it whatever we want, i will set 1 year for my BOTSv1. 

![timesetup](/assets/images/bots-setup/timesetup.png)

Next, add your `competitor` user to Team / Users, navigate to Capture the Flag Admin --> Edit --> Edit Team/Users:
 - Display Username: Whatever you want, mine is Pham Thanh Sang
 - Team: SOC in my Pocket (Whatever you want too)
 - Username(*): this one is the **most important** one, enter your previous `username` that you just created for BOTS Competitor, `phamthanhsang-cs` for me.

Edit your user agreements accept, Admin --> Edit --> Edit Accepted User License Agreements:
 - EulaDateAccepted
 - EulaID
 - EulaName
 - EulaUsername(*): `username` that you just created for BOTS Competitor, mine is `phamthanhsang-cs`

We are so done now, phew ! that was a long run, right ? Let's login Splunk with your `competitor` user, mine is `phamthanhsang-cs` and then go to Capture the Flag app => Questions.

![BOTSV1Question](/assets/images/bots-setup/botsv1questions.png)
_Boss of the SOC version 1 Questions, this is more or less you migh facing in your CTF Challenge_

Let's answer a question to test if we setup our CTF game successfuly. 

![AnswerTestQuestion](/assets/images/bots-setup/answertestquestion.png)
_We are 'blue', right?_

![TestCorrect](/assets/images/bots-setup/testcorrected.png)
_Nice!_

## Troubleshoot Common Issues
- The first issue that we might facing when set things up is `dashboard version is missing` if we plan to use Splunk version 9.x.x.
  - One of the solutions is add `<form version="1.1">` on dashboard XML file, but it might take forever since BOTS have several dashboards, so this method works if we just want to fix some important dashboards.
  - The second i found very helpful is using these scripts and then restart Splunk `systemctl restart splunk` or `$SPLUNK_HOME/bin/splunk restart`: 
```shell
perl -pi -e 's/<form(?=[ >])((?:(?:[^>]| )(?!version="1\.1"))*>)/<form version="1.1"\1/' /opt/splunk/etc/apps/*/*/data/ui/views/*.xml
perl -pi -e 's/<dashboard(?=[ >])((?:(?:[^>]| )(?!version="1\.1"))*>)/<dashboard version="1.1"\1/' /opt/splunk/etc/apps/*/*/data/ui/views/*.xml
```
- Don't have CTF Credentials account to answer question in the CTF app: With this issue, we **must** edit `Edit Team/Users` and `Edit Accepted User License Agreements` like above. 
- There will be several issues relate to system or network connection that you might encounter when setting things up, but that's the most interesting part because it's improving your research, troubleshooting skills.

## Final Words
Setting up BOTS and a self-hosted CTF dashboard may seem challenging at first, but the process is incredibly rewarding. Not only do you gain hands-on experience with the tools and infrastructure, but you also create something entirely your own. By applying the same methodology used in Boss of the SOC version 1, you can set up your own versions of Boss of the SOC 2 and 3.

I hope this guide helps you take that first step toward creating your own environment, whether you're using it for practice or hosting a challenge for others.

If you have any questions or run into any issues, feel free to reach out or leave a comment. I’d love to hear about your experience and help if I can. Thanks for reading, and happy Splunking!
