# Windows11-PerformanceTips

### Ultimate Performance

This Power Configuration speeds up the cpu.

Powershell run as Administrator

Tip: Administrator Powershell Prompt PS C:\Windows\System32> 

Tip: [UserName] Powershell Prompt PS C:\Users\[UserName]>

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

PS C:\Windows\System32> powercfg /setactive 381b4222-f694-41f0-9685-ff5bb260df2e
PS C:\Windows\System32> powercfg /list
```

### CPU Quantum

This is not affected by the PowerCfg.

Tip: Hexadecimal values are always display in the format 0x00000000

Tip: Decimal values are always displayed in the format 0, ie no leading "0x"
```
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\PriorityControl
Win32PrioritySeparation DWORD 0x26  
```
Reboot for changes to take effect.

Desktop: Default 0x26 which provides a short quantum for background processes (31.25 ms) and a longer, boosted quantum for the foreground process (93.75 ms) to maximize apparent responsiveness.

Server: Default 0x18 which provides fixed quantums (187.5 ms) for both foreground and background processes to maximize overall throughput.


Servers run alot of background processes like mail servers, database servers, etc etc, and will very rarely be used for a desktop, so the background quanta (time slice) needs to be bigger and have a small foreground quanta.

Desktops run a desktop where the user is only using a Foreground program, so to make the Foreground process ie the foreground program more responsive, the Foreground quanta (time slice) needs to be bigger.

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
		0x16 	22 			010110 		(01) Long, 		(01) Variable, 			(10) Maximum Foreground boost.
		0x18	24			011000		(01) Long, 		(10) Fixed, 			(00) No Foreground boost. -- Best for Server
		0x2A 	42 			101010 		(10) Short, 	(10) Fixed, 			(10) Maximum Foreground boost.
		0x29	41 			101001		(10) Short, 	(10) Fixed, 			(01) Medium foreground boost.
		0x28 	40			101000		(10) Short, 	(10) Fixed, 			(00) No foreground boost.
		0x26 	38			100110		(10) Short, 	(01) Variable, 			(10) Maximum Foreground boost. -- Best for Desktop Experience
		0x25 	37			100101		(10) Short, 	(01) Variable, 			(01) Medium foreground boost.
		0x24 	36			100100		(10) Short, 	(01) Variable, 			(00) No foreground boost.




### HAGS (Hardware-accelerated GPU Scheduling)

HAGS allows the GPU (Graphics Processing Unit) to manage its own video memory, potentially improving performance by reducing CPU overhead.
It can free up some CPU resources and RAM, potentially boosting FPS (Frames Per Second) by a small margin (1-5%) in some games, but can also help speed up virtualisation software like VMware by offloading graphics to the GPU.

```
Settings > System > Display > Graphics settings
 
Optimisations for Windows Games = On 
```



