---
title: Endpoint Forensics - Amadey @CyberDefender
date: 2025-09-08 00:00:00 +0700
categories: [dfir, blue-teaming, cyber-defender]
tags: [digital-forensics, endpoint-forensics, memory-forensics]
author: <author_id>
description: Amadey Lab @CyberDefender Walkthrough
# toc: false # Uncomment to disable the Table of Contents in the right panel
# comments: false # Uncomment to disable comments
image: /assets/images/preview/cyberdefenders_og.png
---

### Scenario

An after-hours alert from the Endpoint Detection and Response (EDR) system flags suspicious activity on a Windows workstation. The flagged malware aligns with the Amadey Trojan Stealer. Your job is to analyze the presented memory dump and create a detailed report for actions taken by the malware.

### Data Collected
- Windows 7 x64-Snapshot4.vmem ~ 5GB in size

### Tool
- Volatility3 (This Lab provides a dedicated VM so i don't need to install evidences to my host machine)

```bash
ubuntu@ip-172-31-31-204:~/Desktop/Start here/Tools/volatility3$ python3 vol.py -f 'Windows 7 x64-Snapshot4.vmem' windows.info
Volatility 3 Framework 2.5.0
Progress:  100.00		PDB scanning finished
Variable	Value

Kernel Base	0xf80002a55000
DTB	0x187000
Symbols	file:///home/ubuntu/Desktop/Start%20here/Tools/volatility3/volatility3/symbols/windows/ntkrnlmp.pdb/3844DBB920174967BE7AA4A2C20430FA-2.json.xz
Is64Bit	True
IsPAE	False
layer_name	0 WindowsIntel32e
memory_layer	1 FileLayer
KdDebuggerDataBlock	0xf80002c460a0
NTBuildLab	7601.17514.amd64fre.win7sp1_rtm.
CSDVersion	1
KdVersionBlock	0xf80002c46068
Major/Minor	15.7601
MachineType	34404
KeNumberProcessors	1
SystemTime	2023-08-09 21:50:07
NtSystemRoot	C:\Windows
NtProductType	NtProductWinNt
NtMajorVersion	6
NtMinorVersion	1
PE MajorOperatingSystemVersion	6
PE MinorOperatingSystemVersion	1
PE Machine	34404
PE TimeDateStamp	Sat Nov 20 09:30:02 2010
```

### Question

#### Q1: In the memory dump analysis, determining the root of the malicious activity is essential for comprehending the extent of the intrusion. What is the name of the parent process that triggered this malicious behavior?

Still one of my favorite approach when doing "Black Box" forensic which start from network perspective.

```bash
ubuntu@ip-172-31-31-204:~/Desktop/Start here/Tools/volatility3$ python3 vol.py -f 'Windows 7 x64-Snapshot4.vmem' windows.netscan > net.txt
ubuntu@ip-172-31-31-204:~/Desktop/Start here/Tools/volatility3$ cat net.txt
Volatility 3 Framework 2.5.0

Offset	Proto	LocalAddr	LocalPort	ForeignAddr	ForeignPort	State	PID	Owner	Created

0x17d3cf0	TCPv4	-	0	56.123.167.2	0	CLOSED	-	-	N/A
0x1b18870	TCPv4	0.0.0.0	49156	0.0.0.0	0	LISTENING	508	lsass.exe	-
0x1b18870	TCPv6	::	49156	::	0	LISTENING	508	lsass.exe	-
0x22a0c90	UDPv4	0.0.0.0	3702	*	0		368	svchost.exe	2023-08-09 21:32:45.000000
0x251e010	UDPv4	0.0.0.0	3702	*	0		2620	svchost.exe	2023-08-09 21:32:45.000000
0x251e010	UDPv6	::	3702	*	0		2620	svchost.exe	2023-08-09 21:32:45.000000
0x303f820	UDPv4	0.0.0.0	63991	*	0		368	svchost.exe	2023-08-09 21:32:46.000000
0x7347010	UDPv4	0.0.0.0	3702	*	0		368	svchost.exe	2023-08-09 21:32:45.000000
0x7347010	UDPv6	::	3702	*	0		368	svchost.exe	2023-08-09 21:32:45.000000
0x8bcf760	TCPv4	192.168.195.136	139	0.0.0.0	0	LISTENING	4	System	-
0xa1d4d00	UDPv4	127.0.0.1	1900	*	0		2620	svchost.exe	2023-08-09 21:32:17.000000
0xa313010	UDPv6	::1	1900	*	0		2620	svchost.exe	2023-08-09 21:32:17.000000
0xa313950	UDPv4	192.168.195.136	1900	*	0		2620	svchost.exe	2023-08-09 21:32:17.000000
0xe239910	TCPv4	-	0	56.75.178.2	0	CLOSED	2748	lssass.exe	N/A
0x103a6630	TCPv4	0.0.0.0	49155	0.0.0.0	0	LISTENING	500	services.exe	-
0x1610c5b0	UDPv4	0.0.0.0	5355	*	0		920	svchost.exe	2023-08-09 21:50:07.000000
0x1d6a95c0	UDPv4	0.0.0.0	53409	*	0		368	svchost.exe	2023-08-09 21:32:18.000000
0x1d6a95c0	UDPv6	::	53409	*	0		368	svchost.exe	2023-08-09 21:32:18.000000
0x1d6e6590	UDPv4	0.0.0.0	3702	*	0		2620	svchost.exe	2023-08-09 21:32:45.000000
0x1d6e6590	UDPv6	::	3702	*	0		2620	svchost.exe	2023-08-09 21:32:45.000000
0x1d6edd70	UDPv4	0.0.0.0	3702	*	0		2620	svchost.exe	2023-08-09 21:32:45.000000
0x1d757ec0	UDPv4	0.0.0.0	0	*	0		920	svchost.exe	2023-08-09 21:50:07.000000
0x1d757ec0	UDPv6	::	0	*	0		920	svchost.exe	2023-08-09 21:50:07.000000
0x1d75b530	TCPv4	192.168.195.136	49167	41.75.84.12	80	CLOSED	2748	lssass.exe	N/A
0x1d78f480	TCPv4	0.0.0.0	49156	0.0.0.0	0	LISTENING	508	lsass.exe	-
0x1d7be930	UDPv4	0.0.0.0	63992	*	0		368	svchost.exe	2023-08-09 21:32:46.000000
0x1d7be930	UDPv6	::	63992	*	0		368	svchost.exe	2023-08-09 21:32:46.000000
0x1d839800	TCPv4	0.0.0.0	49153	0.0.0.0	0	LISTENING	768	svchost.exe	-
0x1d83ab60	TCPv4	0.0.0.0	49153	0.0.0.0	0	LISTENING	768	svchost.exe	-
0x1d83ab60	TCPv6	::	49153	::	0	LISTENING	768	svchost.exe	-
0x1d8e4c90	UDPv4	0.0.0.0	5355	*	0		920	svchost.exe	2023-08-09 21:47:14.000000
0x1d8e4c90	UDPv6	::	5355	*	0		920	svchost.exe	2023-08-09 21:47:14.000000
0x1d8f2450	UDPv4	0.0.0.0	3702	*	0		368	svchost.exe	2023-08-09 21:32:45.000000
0x1d8f2450	UDPv6	::	3702	*	0		368	svchost.exe	2023-08-09 21:32:45.000000
0x1d90ca70	TCPv4	0.0.0.0	49154	0.0.0.0	0	LISTENING	924	svchost.exe	-
0x1d9a4200	UDPv4	0.0.0.0	53406	*	0		2620	svchost.exe	2023-08-09 21:32:18.000000
0x1d9a5e00	UDPv4	0.0.0.0	3702	*	0		2620	svchost.exe	2023-08-09 21:32:45.000000
0x1d9ac5f0	UDPv4	0.0.0.0	3702	*	0		368	svchost.exe	2023-08-09 21:32:45.000000
0x1daa0720	UDPv4	0.0.0.0	53407	*	0		2620	svchost.exe	2023-08-09 21:32:18.000000
0x1daa0720	UDPv6	::	53407	*	0		2620	svchost.exe	2023-08-09 21:32:18.000000
0x1db7ea10	TCPv4	0.0.0.0	49154	0.0.0.0	0	LISTENING	924	svchost.exe	-
0x1db7ea10	TCPv6	::	49154	::	0	LISTENING	924	svchost.exe	-
0x1dbb3640	TCPv4	0.0.0.0	135	0.0.0.0	0	LISTENING	716	svchost.exe	-
0x1dbb50b0	TCPv4	0.0.0.0	135	0.0.0.0	0	LISTENING	716	svchost.exe	-
0x1dbb50b0	TCPv6	::	135	::	0	LISTENING	716	svchost.exe	-
0x1dbc1010	TCPv4	0.0.0.0	49152	0.0.0.0	0	LISTENING	400	wininit.exe	-
0x1dbc14a0	TCPv4	0.0.0.0	49152	0.0.0.0	0	LISTENING	400	wininit.exe	-
0x1dbc14a0	TCPv6	::	49152	::	0	LISTENING	400	wininit.exe	-
0x1e61eec0	UDPv4	192.168.195.136	53404	*	0		2620	svchost.exe	2023-08-09 21:32:17.000000
0x1e621700	UDPv6	fe80::8d94:2fed:6c71:eac4	1900	*	0		2620	svchost.exe	2023-08-09 21:32:17.000000
0x1e622270	UDPv4	127.0.0.1	53405	*	0		2620	svchost.exe	2023-08-09 21:32:17.000000
0x1e623ec0	UDPv6	fe80::8d94:2fed:6c71:eac4	53402	*	0		2620	svchost.exe	2023-08-09 21:32:17.000000
0x1e673ae0	TCPv4	0.0.0.0	445	0.0.0.0	0	LISTENING	4	System	-
0x1e673ae0	TCPv6	::	445	::	0	LISTENING	4	System	-
0x1e6db500	UDPv4	0.0.0.0	53408	*	0		368	svchost.exe	2023-08-09 21:32:18.000000
0x1e928010	TCPv4	0.0.0.0	49155	0.0.0.0	0	LISTENING	500	services.exe	-
0x1e928010	TCPv6	::	49155	::	0	LISTENING	500	services.exe	-
0x1e94dcf0	TCPv4	192.168.195.136	49168	41.75.84.12	80	CLOSED	2748	lssass.exe	N/A
0x1e9846b0	UDPv4	192.168.195.136	68	*	0		768	svchost.exe	2023-08-09 21:50:07.000000
0x1e988840	UDPv4	0.0.0.0	5355	*	0		920	svchost.exe	2023-08-09 21:50:07.000000
0x1e988840	UDPv6	::	5355	*	0		920	svchost.exe	2023-08-09 21:50:07.000000
0x1ef43830	UDPv6	::1	53403	*	0		2620	svchost.exe	2023-08-09 21:32:17.000000
0x1f1dd690	TCPv4	0.0.0.0	5357	0.0.0.0	0	LISTENING	4	System	-
0x1f1dd690	TCPv6	::	5357	::	0	LISTENING	4	System	-
```

Did you see the anything suspicious in those network connection ? YES IT'S `lssass.exe` at offset `0xe239910`, `0x1d75b530` and `0x1e94dcf0`

```shell
| Offset     | Proto | LocalAddr       | LocalPort | ForeignAddr | ForeignPort | State  | PID  | Owner      | Created |
| ---------- | ----- | --------------- | --------- | ----------- | ----------- | ------ | ---- | ---------- | ------- |
| 0xe239910  | TCPv4 | -               | 0         | 56.75.178.2 | 0           | CLOSED | 2748 | lssass.exe | N/A     |
| 0x1d75b530 | TCPv4 | 192.168.195.136 | 49167     | 41.75.84.12 | 80          | CLOSED | 2748 | lssass.exe | N/A     |
| 0x1e94dcf0 | TCPv4 | 192.168.195.136 | 49168     | 41.75.84.12 | 80          | CLOSED | 2748 | lssass.exe | N/A     |
```

Since the malicious process has been detected, with it's PID, external IP Address, now i can move to process analysis, to detect entire process tree of this process, `Vol3` support a module called `windows.pstree`

```shell
ubuntu@ip-172-31-31-204:~/Desktop/Start here/Tools/volatility3$ python3 vol.py -f 'Windows 7 x64-Snapshot4.vmem' windows.pstree --pid 2748
Volatility 3 Framework 2.5.0
Progress:  100.00		PDB scanning finished
PID	PPID	ImageFileName	Offset(V)	Threads	Handles	SessionId	Wow64	CreateTime	ExitTime

2748	2524	lssass.exe	0xfa800300a750	7	254	1	True	2023-08-09 21:33:04.000000 	N/A
* 3064	2748	rundll32.exe	0xfa8003042b30	1	64	1	True	2023-08-09 21:33:56.000000 	N/A
```

Look into this process tree, i could determine malicious process that trigger EDR, and, answer for this question.

> ANSWER: `lssass.exe`
{: .prompt-info }

#### Q2: Once the rogue process is identified, its exact location on the device can reveal more about its nature and source. Where is this process housed on the workstation?

Similar like Task Manager (built-in Windows utility), you could the processes and it's execution path, Vol3 support module named `windows.cmdline`, malicious process might be able to extract by the same methodology.

![pic1](assets/images/cyberdefender/amadey/pic1.png)

```shell
ubuntu@ip-172-31-31-204:~/Desktop/Start here/Tools/volatility3$ python3 vol.py -f 'Windows 7 x64-Snapshot4.vmem' windows.cmdline --pid 2748
Volatility 3 Framework 2.5.0
Progress:  100.00		PDB scanning finished
PID	Process	Args

2748	lssass.exe	"C:\Users\0XSH3R~1\AppData\Local\Temp\925e7e99c5\lssass.exe"
```

This rogue process came from user `temp` folder.

> ANSWER: `C:\Users\0XSH3R~1\AppData\Local\Temp\925e7e99c5\lssass.exe`
{: .prompt-info }

#### Q3: Persistent external communications suggest the malware's attempts to reach out C2C server. Can you identify the Command and Control (C2C) server IP that the process interacts with?




#### Q4: Following the malware link with the C2C, the malware is likely fetching additional tools or modules. How many distinct files is it trying to bring onto the compromised workstation?



```shell
ubuntu@ip-172-31-31-204:~/Desktop/Start here/Tools/volatility3$ python3 vol.py -f 'Windows 7 x64-Snapshot4.vmem' -o lssass/ windows.dumpfiles --pid 2748
Volatility 3 Framework 2.5.0
Progress:  100.00		PDB scanning finished
Cache	FileObject	FileName	Result

DataSectionObject	0xfa8001d44c00	index.dat	file.0xfa8001d44c00.0xfa80030e7260.DataSectionObject.index.dat.dat
DataSectionObject	0xfa8002adda70	index.dat	file.0xfa8002adda70.0xfa8002cf1b00.DataSectionObject.index.dat.dat
DataSectionObject	0xfa8002fe27b0	index.dat	file.0xfa8002fe27b0.0xfa8001d85010.DataSectionObject.index.dat.dat
ImageSectionObject	0xfa8001fb4340	apisetschema.dll	file.0xfa8001fb4340.0xfa8001fb8010.ImageSectionObject.apisetschema.dll.img
DataSectionObject	0xfa8001d44c00	index.dat	file.0xfa8001d44c00.0xfa80030e7260.DataSectionObject.index.dat.dat
DataSectionObject	0xfa8002fe27b0	index.dat	file.0xfa8002fe27b0.0xfa8001d85010.DataSectionObject.index.dat.dat
DataSectionObject	0xfa8002adda70	index.dat	file.0xfa8002adda70.0xfa8002cf1b00.DataSectionObject.index.dat.dat
DataSectionObject	0xfa8002d81310	cversions.2.db	file.0xfa8002d81310.0xfa8002d81430.DataSectionObject.cversions.2.db.dat
DataSectionObject	0xfa8002d81310	cversions.2.db	file.0xfa8002d81310.0xfa8002d81430.DataSectionObject.cversions.2.db.dat
DataSectionObject	0xfa8003119290	lssass.exe	file.0xfa8003119290.0xfa8001ef4b70.DataSectionObject.lssass.exe.dat
ImageSectionObject	0xfa8003119290	lssass.exe	file.0xfa8003119290.0xfa8001d2eac0.ImageSectionObject.lssass.exe.img
DataSectionObject	0xfa8002efadc0	oleaccrc.dll	Error dumping file
DataSectionObject	0xfa8002d81310	cversions.2.db	file.0xfa8002d81310.0xfa8002d81430.DataSectionObject.cversions.2.db.dat
ImageSectionObject	0xfa800208bbc0	sspicli.dll	file.0xfa800208bbc0.0xfa800208b010.ImageSectionObject.sspicli.dll.img
ImageSectionObject	0xfa8002d4d070	propsys.dll	file.0xfa8002d4d070.0xfa8002dca6d0.ImageSectionObject.propsys.dll.img
ImageSectionObject	0xfa8002afa220	nlaapi.dll	file.0xfa8002afa220.0xfa80030fc010.ImageSectionObject.nlaapi.dll.img
ImageSectionObject	0xfa8001404a20	wship6.dll	Error dumping file
ImageSectionObject	0xfa8002078460	FWPUCLNT.DLL	Error dumping file
ImageSectionObject	0xfa800d5fe6d0	mswsock.dll	file.0xfa800d5fe6d0.0xfa8002d7e260.ImageSectionObject.mswsock.dll.img
ImageSectionObject	0xfa8001ee7070	wshbth.dll	Error dumping file
ImageSectionObject	0xfa8001e2d340	WSHTCPIP.DLL	Error dumping file
ImageSectionObject	0xfa800308aa20	winrnr.dll	Error dumping file
ImageSectionObject	0xfa8002f5bf20	NapiNSP.dll	Error dumping file
ImageSectionObject	0xfa8002d071e0	pnrpnsp.dll	Error dumping file
ImageSectionObject	0xfa8002da52c0	rasadhlp.dll	Error dumping file
ImageSectionObject	0xfa8002c1eae0	rasman.dll	Error dumping file
ImageSectionObject	0xfa8001daaf20	SensApi.dll	Error dumping file
ImageSectionObject	0xfa8002d70600	IPHLPAPI.DLL	file.0xfa8002d70600.0xfa8002e7b5c0.ImageSectionObject.IPHLPAPI.DLL.img
ImageSectionObject	0xfa8002d5ddc0	rasapi32.dll	Error dumping file
ImageSectionObject	0xfa8002e08700	ieframe.dll	Error dumping file
ImageSectionObject	0xfa8002fbddd0	profapi.dll	file.0xfa8002fbddd0.0xfa8003085810.ImageSectionObject.profapi.dll.img
ImageSectionObject	0xfa80015e16c0	winmm.dll	file.0xfa80015e16c0.0xfa8002ed8930.ImageSectionObject.winmm.dll.img
ImageSectionObject	0xfa8002fbbd30	AcGenral.dll	file.0xfa8002fbbd30.0xfa8002a8e1a0.ImageSectionObject.AcGenral.dll.img
ImageSectionObject	0xfa8002b41440	dnsapi.dll	file.0xfa8002b41440.0xfa8001d26190.ImageSectionObject.dnsapi.dll.img
ImageSectionObject	0xfa8002f5add0	rtutils.dll	Error dumping file
ImageSectionObject	0xfa8002bda240	winnsi.dll	file.0xfa8002bda240.0xfa8001761e60.ImageSectionObject.winnsi.dll.img
ImageSectionObject	0xfa80015fddd0	dwmapi.dll	file.0xfa80015fddd0.0xfa8002daee60.ImageSectionObject.dwmapi.dll.img
ImageSectionObject	0xfa8002fe3d20	ntmarta.dll	file.0xfa8002fe3d20.0xfa8002d24340.ImageSectionObject.ntmarta.dll.img
ImageSectionObject	0xfa8002ad1330	oleacc.dll	file.0xfa8002ad1330.0xfa80030e9500.ImageSectionObject.oleacc.dll.img
ImageSectionObject	0xfa8001f0ac40	msacm32.dll	file.0xfa8001f0ac40.0xfa8002d41370.ImageSectionObject.msacm32.dll.img
ImageSectionObject	0xfa8002d4b070	sfc.dll	file.0xfa8002d4b070.0xfa8003057300.ImageSectionObject.sfc.dll.img
ImageSectionObject	0xfa8002d586c0	mpr.dll	file.0xfa8002d586c0.0xfa8002bf60d0.ImageSectionObject.mpr.dll.img
ImageSectionObject	0xfa800ddd39e0	uxtheme.dll	file.0xfa800ddd39e0.0xfa8002b399b0.ImageSectionObject.uxtheme.dll.img
ImageSectionObject	0xfa80105fede0	sfc_os.dll	file.0xfa80105fede0.0xfa8002d44450.ImageSectionObject.sfc_os.dll.img
ImageSectionObject	0xfa8002fc2650	winspool.drv	Error dumping file
ImageSectionObject	0xfa800303e870	comctl32.dll	file.0xfa800303e870.0xfa8002ed7790.ImageSectionObject.comctl32.dll.img
ImageSectionObject	0xfa8002f21c80	wow64cpu.dll	file.0xfa8002f21c80.0xfa8002f21a40.ImageSectionObject.wow64cpu.dll.img
ImageSectionObject	0xfa800af76c10	AcLayers.dll	file.0xfa800af76c10.0xfa80015ed4b0.ImageSectionObject.AcLayers.dll.img
ImageSectionObject	0xfa8003044370	version.dll	file.0xfa8003044370.0xfa8003039c80.ImageSectionObject.version.dll.img
ImageSectionObject	0xfa8002d50c90	userenv.dll	file.0xfa8002d50c90.0xfa8003039aa0.ImageSectionObject.userenv.dll.img
ImageSectionObject	0xfa8002c15070	samcli.dll	file.0xfa8002c15070.0xfa8002faec80.ImageSectionObject.samcli.dll.img
ImageSectionObject	0xfa8002c32070	apphelp.dll	file.0xfa8002c32070.0xfa8002d6ed20.ImageSectionObject.apphelp.dll.img
ImageSectionObject	0xfa8002e07420	wow64.dll	file.0xfa8002e07420.0xfa8002ee7cf0.ImageSectionObject.wow64.dll.img
ImageSectionObject	0xfa8001d3e260	wow64win.dll	file.0xfa8001d3e260.0xfa8002f5dc40.ImageSectionObject.wow64win.dll.img
ImageSectionObject	0xfa800208b850	cryptbase.dll	file.0xfa800208b850.0xfa800208b2d0.ImageSectionObject.cryptbase.dll.img
ImageSectionObject	0xfa8002083530	kernel32.dll	file.0xfa8002083530.0xfa800843c730.ImageSectionObject.kernel32.dll.img
ImageSectionObject	0xfa8002084b60	msvcrt.dll	file.0xfa8002084b60.0xfa8001e07d10.ImageSectionObject.msvcrt.dll.img
ImageSectionObject	0xfa800f0c45c0	usp10.dll	file.0xfa800f0c45c0.0xfa80020894a0.ImageSectionObject.usp10.dll.img
ImageSectionObject	0xfa8002086b60	imm32.dll	file.0xfa8002086b60.0xfa8002086650.ImageSectionObject.imm32.dll.img
ImageSectionObject	0xfa8002086860	nsi.dll	file.0xfa8002086860.0xfa800208ac50.ImageSectionObject.nsi.dll.img
ImageSectionObject	0xfa800208a7f0	wininet.dll	file.0xfa800208a7f0.0xfa800208bcb0.ImageSectionObject.wininet.dll.img
ImageSectionObject	0xfa8002086560	msctf.dll	file.0xfa8002086560.0xfa8002086130.ImageSectionObject.msctf.dll.img
ImageSectionObject	0xfa80020893b0	KernelBase.dll	file.0xfa80020893b0.0xfa8002086950.ImageSectionObject.KernelBase.dll.img
ImageSectionObject	0xfa8002084750	oleaut32.dll	file.0xfa8002084750.0xfa8001e076d0.ImageSectionObject.oleaut32.dll.img
ImageSectionObject	0xfa8002085070	iertutil.dll	file.0xfa8002085070.0xfa8002085af0.ImageSectionObject.iertutil.dll.img
ImageSectionObject	0xfa8001e07330	Wldap32.dll	file.0xfa8001e07330.0xfa80020899f0.ImageSectionObject.Wldap32.dll.img
ImageSectionObject	0xfa8001e075e0	psapi.dll	file.0xfa8001e075e0.0xfa8001e07120.ImageSectionObject.psapi.dll.img
ImageSectionObject	0xfa8002084450	user32.dll	file.0xfa8002084450.0xfa8001e07a10.ImageSectionObject.user32.dll.img
ImageSectionObject	0xfa8002084e60	cfgmgr32.dll	file.0xfa8002084e60.0xfa8002084540.ImageSectionObject.cfgmgr32.dll.img
ImageSectionObject	0xfa800843c2d0	crypt32.dll	file.0xfa800843c2d0.0xfa800207f8a0.ImageSectionObject.crypt32.dll.img
ImageSectionObject	0xfa8002080070	ole32.dll	file.0xfa8002080070.0xfa8002080cd0.ImageSectionObject.ole32.dll.img
ImageSectionObject	0xfa800207f500	rpcrt4.dll	file.0xfa800207f500.0xfa8002080320.ImageSectionObject.rpcrt4.dll.img
ImageSectionObject	0xfa8002080560	devobj.dll	file.0xfa8002080560.0xfa8002084c50.ImageSectionObject.devobj.dll.img
ImageSectionObject	0xfa800207f7b0	msasn1.dll	file.0xfa800207f7b0.0xfa800207f2f0.ImageSectionObject.msasn1.dll.img
ImageSectionObject	0xfa800843c940	shell32.dll	Error dumping file
ImageSectionObject	0xfa800843c640	urlmon.dll	file.0xfa800843c640.0xfa800207f010.ImageSectionObject.urlmon.dll.img
ImageSectionObject	0xfa800207ec20	gdi32.dll	file.0xfa800207ec20.0xfa800207e8c0.ImageSectionObject.gdi32.dll.img
ImageSectionObject	0xfa800207ead0	clbcatq.dll	file.0xfa800207ead0.0xfa800843ce60.ImageSectionObject.clbcatq.dll.img
ImageSectionObject	0xfa80020838a0	shlwapi.dll	file.0xfa80020838a0.0xfa8002083320.ImageSectionObject.shlwapi.dll.img
ImageSectionObject	0xfa8002083f20	sechost.dll	file.0xfa8002083f20.0xfa8002083010.ImageSectionObject.sechost.dll.img
ImageSectionObject	0xfa800207cd70	advapi32.dll	file.0xfa800207cd70.0xfa800207ca00.ImageSectionObject.advapi32.dll.img
ImageSectionObject	0xfa800207c910	setupapi.dll	file.0xfa800207c910.0xfa800207dc40.ImageSectionObject.setupapi.dll.img
ImageSectionObject	0xfa800207db50	ws2_32.dll	file.0xfa800207db50.0xfa800207ed10.ImageSectionObject.ws2_32.dll.img
ImageSectionObject	0xfa800203c930	lpk.dll	file.0xfa800203c930.0xfa800207ce60.ImageSectionObject.lpk.dll.img
ImageSectionObject	0xfa800162dac0	ntdll.dll	file.0xfa800162dac0.0xfa8001e2cad0.ImageSectionObject.ntdll.dll.img
ImageSectionObject	0xfa800162df20	ntdll.dll	file.0xfa800162df20.0xfa800162bac0.ImageSectionObject.ntdll.dll.img
```

```shell
ubuntu@ip-172-31-31-204:~/Desktop/Start here/Tools/volatility3$ strings --bytes 16 lssass/file.0xfa8003119290.0xfa8001d2eac0.ImageSectionObject.lssass.exe.img
!This program cannot be run in DOS mode.
SleepConditionVariableCS
WakeAllConditionVariable
`local static guard'
`vbase destructor'
`vector deleting destructor'
`default constructor closure'
`scalar deleting destructor'
`vector constructor iterator'
`vector destructor iterator'
`vector vbase constructor iterator'
`virtual displacement map'
`eh vector constructor iterator'
`eh vector destructor iterator'
`eh vector vbase constructor iterator'
`copy constructor closure'
`local vftable constructor closure'
`placement delete closure'
`placement delete[] closure'
`managed vector constructor iterator'
`managed vector destructor iterator'
`eh vector copy constructor iterator'
`eh vector vbase copy constructor iterator'
`dynamic initializer for '
`dynamic atexit destructor for '
`vector copy constructor iterator'
`vector vbase copy constructor iterator'
`managed vector copy constructor iterator'
`local static thread guard'
operator co_await
 Type Descriptor'
 Base Class Descriptor at (
 Base Class Array'
 Class Hierarchy Descriptor'
 Complete Object Locator'
`anonymous namespace'
InitializeCriticalSectionEx
LocaleNameToLCID
AppPolicyGetProcessTerminationMethod
dddd, MMMM dd, yyyy
 !"#$%&'()*+,-./0123456789:;<=>?@abcdefghijklmnopqrstuvwxyz[\]^_`abcdefghijklmnopqrstuvwxyz{|}~
 !"#$%&'()*+,-./0123456789:;<=>?@ABCDEFGHIJKLMNOPQRSTUVWXYZ[\]^_`ABCDEFGHIJKLMNOPQRSTUVWXYZ{|}~
Unknown exception
bad array new length
invalid stoi argument
stoi argument out of range
ada76b8b0e1f6892ee93c20ab8946117
116711e5a2ab05704bbb0c589b88924d
5e995b05dff403d693a5b7bc37af269e
NCcqJUBvCn7zIgC=
L6JrT3pw00VpUUelgKfo
OWIXUUamDXboIN==
cJRd9HSAAqL9UN==
L3NuUXC1OWqAPXK8XYjGXMGFJz7GOoqtIF9QOjx=
U39CQGaCJoLhOTjagq7rdTO0YCbi7cHrd6N9L4Sz2qLz9CXcgrLhdTUcVkTnOSVfZT==
U39CQGaCJoLhOTjagq7rdTO0YCbi7cHrd6N9L4Sz2qLz9CXcgrLhdTUcSUfp7wZuZ0J9QYKm2mrY6DTjfGzydTmk0UHs
Y51gDD6ExJzSMBjJMG7rNUtg
U39CQGaCJoLhOTjagq7rdTO0YCbi7cHrd6N9L4Sz2qLz9CXcgrLhdTUcVkTn
IF9AUXum3KKlGYPFMGG=
U39CQGaCJoLhOTjagq7rdTO0YCbi7cHrd6N9L4Sz2qLz9CXcgrLhdTUcSUfp7wZuZ0J9P3em1KmlMj7j3KTqen==
JYVPMWGRJoZLNRvwNU==
Y6JhUD2l1KncTZvggG3cdxm8
c5hh7HtABmVp7Du=
a5Vu7nStBDyzUDvj
R5VWOnC101PqP0jqhKTlUNUmct==
UKJrU4Gi1YHm9DD4
QYZxP2NhJ6Zr9Eb9gqS=
S5Fv8HSz26j DBv92k==
UJFqUHBhJ6Lo9UHghLi=
RJ9f9H6zxJ4qTd==
MCYsQH61N0nYUTLSgqjSgH==
QplWUHSnO0VpUUG=
V5lqMHSnO0VpUUG=
MGEuHAN2Cn39JN==
Q59q9HSv3GRZ UzcSmzlfNm0bUza8dGrZp9u7TYlN1HmJvz f7Tm1xuyfQYmGIQp
DTpz7321O0V5GRPgg7zneTa0bT7nJ9rib6JpGXOi3KubDD39f0S1KdGheDCbJoriaZxh7nCuOXQn
Ij0GL36v3KLz9zZLi1zdQ qhdEzl6MDddJlr7j6wN7Hq9zZqhLHd0NPNDdYD
Q59q9HSv3GRZ UzcSmz eyqsbTLa9wbrbl90GYa43CRr70HkP1TqdxKuZZ7dUMG=
U4lPQFSOMID68kHcfrPvdTU0dj7lPSLWXHNr7oOz16nhLZ7kgLTS1OyOZTZeSuDrb0BX9HSzIqvyUN==
Q59p8IS1O1zTTTZc
YZJfUHSnO68u6jrjf03neyuyd0PU9d40e0osHUFACHK7IweWPZ6=
U4lPQFSOMID68kHcfrPvdTU0dj7lPSLWXHNr7oOz16nhQT3ghKTcXdak0T76LQZKVIJLOGuXHYHKOYu=
U4lPQFSOMIDA7kPpf6vL1OFwNAD6PSLudplfUYKdFqvE6TLve1Lodxu5YCXiUwLr
RJViTYSt3JDq9EPgfqbrNc7S0ULo7xLWaZ9q
RJViTYSt3JDq9EPgfqbrNcaS0ULo7xLWaZ9q
U39CQGaCJoLhOTjagq7rdTO0YCbi7cHrd6McOmOdF7LD8jTlhJXdeeCpcZ2=
UKJrUISk3IVm7TS=
Q6Vu8nSv3Iz66Tvb
ZZNk7zy05IDGLXvKMGG=
cqVqUHutBDyzUUfc
IqRd83qs00nxDz7dMG7hdJpi
IlAiEjy100Rq70TRMHC9L Ng0DTlDq==
UJ9ZUYGA0KLx7z3ciKS=
LZV0UXK23KbA7kzmfKjbgJqy0TZo9wLvaZdqUXNhAYPu7DS8Mk==
abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789
NtUnmapViewOfSection
invalid string position
D:\Mktmp\Amadey\Release\Amadey.pdb
GetModuleFileNameA
WriteProcessMemory
GetComputerNameExW
GetModuleHandleA
SetCurrentDirectoryA
GetFileAttributesA
GetThreadContext
RemoveDirectoryA
ReadProcessMemory
CreateDirectoryA
SetThreadContext
GetSidIdentifierAuthority
LookupAccountNameA
GetSidSubAuthority
GetSidSubAuthorityCount
RegQueryValueExA
SHGetFolderPathA
SHFileOperationA
HttpOpenRequestA
InternetOpenUrlA
InternetCloseHandle
HttpSendRequestA
InternetConnectA
InternetReadFile
EnterCriticalSection
LeaveCriticalSection
InitializeCriticalSectionAndSpinCount
DeleteCriticalSection
WaitForSingleObjectEx
GetModuleHandleW
UnhandledExceptionFilter
SetUnhandledExceptionFilter
GetCurrentProcess
TerminateProcess
IsProcessorFeaturePresent
IsDebuggerPresent
QueryPerformanceCounter
GetCurrentProcessId
GetCurrentThreadId
GetSystemTimeAsFileTime
InitializeSListHead
GetModuleHandleExW
GetFileInformationByHandle
SystemTimeToTzSpecificLocalTime
FileTimeToSystemTime
GetModuleFileNameW
MultiByteToWideChar
GetCurrentDirectoryW
GetFullPathNameW
FindFirstFileExW
WideCharToMultiByte
GetEnvironmentStringsW
FreeEnvironmentStringsW
SetEnvironmentVariableW
FlushFileBuffers
GetTimeZoneInformation
SetFilePointerEx

abcdefghijklmnopqrstuvwxyz
ABCDEFGHIJKLMNOPQRSTUVWXYZ

abcdefghijklmnopqrstuvwxyz
ABCDEFGHIJKLMNOPQRSTUVWXYZ
.?AVinvalid_argument@std@@
.?AVlogic_error@std@@
.?AVlength_error@std@@
.?AVout_of_range@std@@
.?AVbad_alloc@std@@
.?AVexception@std@@
.?AVbad_array_new_length@std@@
<?xml version='1.0' encoding='UTF-8' standalone='yes'?>
<assembly xmlns='urn:schemas-microsoft-com:asm.v1' manifestVersion='1.0'>
  <trustInfo xmlns="urn:schemas-microsoft-com:asm.v3">
      <requestedPrivileges>
        <requestedExecutionLevel level='asInvoker' uiAccess='false' />
      </requestedPrivileges>
```

#### Q5: Identifying the storage points of these additional components is critical for containment and cleanup. What is the full path of the file downloaded and used by the malware in its malicious activity?

#### Q6: Once retrieved, the malware aims to activate its additional components. Which child process is initiated by the malware to execute these files?

#### Q7: Understanding the full range of Amadey's persistence mechanisms can help in an effective mitigation. Apart from the locations already spotlighted, where else might the malware be ensuring its consistent presence?

