# Windows11-PerformanceTips

There are major differences between windows 11 24H2 and 25H2.  

### Which program bitness to download

Where possible its generally best to download the 32bit version because these are still faster than 64bit programs and are generally smaller in file size and this is especially important if the desktop computer or laptop is a low spec machine.

If you are doing some serious computing, like running SQL server, an Ai, or virtualisation software, then a 64bit version is best in order to take advantage of extra RAM.

### TLDR
In order of what is most noticable on slow computers.

CPU Quantum 
Windows Special Effects.


### Ultimate Performance

This Power Configuration keeps hardware active/awake for longer, so the lag experienced whilst parts of the computer hardware come alive, is removed. 

Using the below will hide options seen in ```Settings > System > Power & Battery > Power Mode``` but can be seen in ```Control Panel > Power Options > Choose or Customize a power plan```.

Powershell run as Administrator

Tip: Administrator Powershell Prompt ```PS C:\Windows\System32>``` 

Tip: [UserName] Powershell Prompt ```PS C:\Users\[UserName]>```

```
PS C:\Windows\System32> powercfg /list
Existing Power Schemes (* Active)
-----------------------------------
Power Scheme GUID: 381b4222-f694-41f0-9685-ff5bb260df2e  (Balanced) *
PS C:\Windows\System32>

PS C:\Windows\System32> powercfg -duplicatescheme e9a42b02-d5df-448d-aa00-03f14749eb61
Power Scheme GUID: 3ea0e233-0610-4c4e-96f0-0644141592ba  (Ultimate Performance)
PS C:\Windows\System32>
PS C:\Windows\System32> powercfg /setactive 3ea0e233-0610-4c4e-96f0-0644141592ba
PS C:\Windows\System32> powercfg /list
Existing Power Schemes (* Active)
-----------------------------------
Power Scheme GUID: 381b4222-f694-41f0-9685-ff5bb260df2e  (Balanced)
Power Scheme GUID: 3ea0e233-0610-4c4e-96f0-0644141592ba  (Ultimate Performance) *
PS C:\Windows\System32>
```

### CPU Quantum

Altering this will see the best performance gains for even the slowest most basic of desktop or laptop computers.

Tip: Hexadecimal values are always display in the format ```0x00000000``` or lead with ```0x```

Tip: Decimal values are always displayed in the format ```0```, ie no leading ```0x```
```
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\PriorityControl
Win32PrioritySeparation DWORD 0x26  
```
Reboot for changes to take effect.

Desktop: Default 0x26 which provides a short quantum for background processes (31.25 ms) and a longer, boosted quantum for the foreground process (93.75 ms) to maximize apparent responsiveness.

Server: Default 0x18 which provides fixed quantums (187.5 ms) for both foreground and background processes to maximize overall throughput.


Servers run alot of background processes like mail servers, database servers, etc etc, and will very rarely be used for a desktop, so the background quanta (time slice) needs to be bigger and have a small foreground quanta.

Desktops run a desktop where the user is only using a Foreground program at any one time, although they might have more than one program open, so to make the Foreground process ie the foreground program more responsive, the Foreground quanta (time slice) needs to be bigger.

This value consists of 6 bits divided into the three 2-bit fields 
```
		Short vs Long | Variables vs Fixed | Foreground Boost |
Bit 		5,4					3,2					1,0
```		
```
Short vs Long (Bits 4 & 5)

	1 = specifies long quanta
	
	2 = specifies short quanta
	
Variable vs Fixed

	1 = vary the quantum for the foreground process
	
	2 = quantum values dont vary for the foreground process
	
Foreground Boost

	0 = No boost
	
	1 = Medium boost
	
	2 = Maximum boost
	
	
Decimal to two bits

	0 = 00
	
	1 = 01
	
	2 = 10
```

To Calculate 
	
	To make the desktop program as responsive as possible.
	
	Short vs Long = Long Quanta = 1 = 01
	
	Variable vs Fixed = Variable Foreground 1 = 01
	
	Foreground Boost = Maximum boost = 2 = 10
	
	= 010110 = 22 = 0x16

		Hex 	Decimal		Binary		Short vs Long, 	Variable vs Fixed, 		Foreground Boost  
		0x16 	22 			010110 		(01) Long, 		(01) Variable, 			(10) Maximum Foreground boost. -- Best for Desktop Experience**
		0x18	24			011000		(01) Long, 		(10) Fixed, 			(00) No Foreground boost. -- Best for Server
		0x2A 	42 			101010 		(10) Short, 	(10) Fixed, 			(10) Maximum Foreground boost.
		0x29	41 			101001		(10) Short, 	(10) Fixed, 			(01) Medium foreground boost.
		0x28 	40			101000		(10) Short, 	(10) Fixed, 			(00) No foreground boost.
		0x26 	38			100110		(10) Short, 	(01) Variable, 			(10) Maximum Foreground boost. -- Best for Desktop Experience 
		0x25 	37			100101		(10) Short, 	(01) Variable, 			(01) Medium foreground boost.
		0x24 	36			100100		(10) Short, 	(01) Variable, 			(00) No foreground boost.

** If you have other desktop programs open and they are running in the background doing stuff (technical term) like calculating or compiling, this could be better for them, but may reduce the responsiveness of the Foreground process ie the program with focus being worked in.


### HAGS (Hardware-accelerated GPU Scheduling)

Performance gains are not noticeable, but could benefit some Gamers.

HAGS allows the GPU (Graphics Processing Unit) to manage its own video memory, potentially improving performance by reducing CPU overhead.
It can free up some CPU resources and RAM, potentially boosting FPS (Frames Per Second) by a small margin (1-5%) in some games, but can also help speed up virtualisation software like VMware by offloading graphics to the GPU. 

```
Settings > System > Display > Graphics settings
 
Optimisations for Windows Games = On 
```

The following Registry key appears to have no effect on the HAGS setting using the Settings route shown above. It is suggested on some other websites though, so probably applies to an earlier version of Windows, eg Windows 10 or Windows 23H2. YMMV.

```
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\GraphicsDrivers
HwSchMode DWORD controls it (0=On, 1=Off, 2=Default/Auto)
```

### Notepad++ run multiple instances for Alt-Tabbing purposes

	```
	Notepad++ > Settings > Preferences > Multi-Instance & Date > Always in multi-instance mode
	```

### Vmware using Intel VT-x/EPT or AMD-V/RVI

Some of the windows security measures and Linux on Window (WSL) uses Microsoft's Hyper-V virtualisation platform. 

This can reduce performance in Windows when using virtualisation software like VMware Workstation because it has to use MS Hyper-V and not its own code.

These instructions below will disable and remove Hyper-V so that Vmware Workstation can use the VM Guest option, Processors > Virtualisation Engine > Use Intel VT-x/EPT or AMD-V/RVI > Ticked.

Windows 11 24H2 has a "bug" which prevents Hyper-V from being disabled, so the final step below has to be performed.

These steps will reduce the built-in security measures Microsoft has built into Windows 11 because they use Hyper-V, but for low performance machines, disabling this can help speed things up a bit. If using Windows Defender/Security and not an external AntiVirus solution, switching off some of the Windows Defender/Security options which connects to the internet will also speed up the responsiveness of programs. 

```
Load services.msc 
Stop (right mouse click on the service and choose stop) all services with Hyper-V in the name and "HV Host Service".
```
```
Features - Windows Features 
Uncheck Hyper-V (including all sub-items),  Windows Hypervisor Platform, & (WSL) Linux subsystem for windows.
Reboot 
```

Launch Powershell as Administrator
```
PS C:\Windows\system32> bcdedit /set hypervisorlaunchtype off
The operation completed successfully.
PS C:\Windows\system32>
```

If the machine is running Group Policy.
```
Local Computer Policy > Computer Configuration > Administrative Templates > System > Device Guard
Double-click on "Turn on Virtualization Based Security".
Select Disabled and click OK.
```

In the search box type ```coreisolation```.
Switch off "Memory Integrity".


For Win11 24H2 perform this step.
Download the Device Guard and Credential Guard hardware readiness tool from https://www.microsoft.com/en-us/download/details.aspx?id=53337

Extract the zip file contents.
Launch Powershell as Administrator
```
PS C:\Windows\system32> Set-ExecutionPolicy Unrestricted -Scope Process

Execution Policy Change
The execution policy helps protect you from scripts that you do not trust. Changing the execution policy might expose
you to the security risks described in the about_Execution_Policies help topic at
https:/go.microsoft.com/fwlink/?LinkID=135170. Do you want to change the execution policy?
[Y] Yes  [A] Yes to All  [N] No  [L] No to All  [S] Suspend  [?] Help (default is "N"): a
PS C:\Windows\system32>
```

```
PS C:\Windows\system32> cd C:\Users\[UserName]\Downloads\dgreadiness_v3.6
PS C:\Users\[UserName]\Downloads\dgreadiness_v3.6> cd dgreadiness_v3.6
PS C:\Users\[UserName]\Downloads\dgreadiness_v3.6\dgreadiness_v3.6> .\DG_Readiness_Tool_v3.6.ps1 -Disable

Security warning
Run only scripts that you trust. While scripts from the internet can be useful, this script can potentially harm your
computer. If you trust this script, use the Unblock-File cmdlet to allow the script to run without this warning
message. Do you want to run C:\Users\[UserName]\Downloads\dgreadiness_v3.6\dgreadiness_v3.6\DG_Readiness_Tool_v3.6.ps1?
[D] Do not run  [R] Run once  [S] Suspend  [?] Help (default is "D"): r
###########################################################################
Readiness Tool Version 3.4 Release.
Tool to check if your device is capable to run Device Guard and Credential Guard.
###########################################################################
Disabling Device Guard and Credential Guard
Deleting RegKeys to disable DG/CG
Disabling Hyper-V and IOMMU
Disabling Hyper-V and IOMMU successful

Please reboot the machine, for settings to be applied.
PS C:\Users\[UserName]\Downloads\dgreadiness_v3.6\dgreadiness_v3.6>
```

Reboot the machine.

Launch Powershell as Administrator
```
PS C:\Windows\system32> bcdedit /set hypervisorlaunchtype off
The operation completed successfully.
PS C:\Windows\system32>
```

### Windows Special Effects

If you want snappy instant windows without the special animation effects like fading in and out, and other effects, switch off the effects found in 
```Settings > Accessibility > Visual Effects``` 

or to switch them all off for maximum Performance
Press Windows Key and R, to get the Windows Run window up, and paste in ```SystemPropertiesPerformance.exe``` and run it. 
A new window titled Performance Options will appear where you can select ```Adjust for Best Performance```. Tick Adjust for Best Performance, all options will become unticked.
Tick the option Smooth Edges of screen Fonts. This option will make webpages and desktop apps appear properly.

