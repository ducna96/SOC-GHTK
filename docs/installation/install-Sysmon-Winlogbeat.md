
Contact Info: ducna36@ghtk.co
## Sysmon:
1. Introduction
System Monitor (Sysmon) is a Windows system service and device driver that, once installed on a system, remains resident across system reboots to monitor and log system activity to the Windows event log. It provides detailed information about process creations, network connections, and changes to file creation time. By collecting the events it generates using Windows Event Collection or SIEM agents and subsequently analyzing them, you can identify malicious or anomalous activity and understand how intruders and malware operate on your network.
Note that Sysmon does not provide analysis of the events it generates, nor does it attempt to protect or hide itself from attackers.
2.	Overview of Sysmon Capabilities
	-	Logs process creation with full command line for both current and parent processes.
	-	Records the hash of process image files using SHA1 (the default), MD5, SHA256 or IMPHASH.
	-	Multiple hashes can be used at the same time.
	-	Includes a process GUID in process create events to allow for correlation of events even when Windows reuses process IDs.
	-	Includes a session GUID in each event to allow correlation of events on same logon session.
	-	Logs loading of drivers or DLLs with their signatures and hashes.
	-	Logs opens for raw read access of disks and volumes.
	-	Optionally logs network connections, including each connection’s source process, IP addresses, port numbers, hostnames and port names.
	-	Detects changes in file creation time to understand when a file was really created. Modification of file create timestamps is a technique commonly used by malware to cover its tracks.
	-	Automatically reload configuration if changed in the registry.
	-	Rule filtering to include or exclude certain events dynamically.
	-	Generates events from early in the boot process to capture activity made by even sophisticated kernel-mode malware.

3. Install Sysmon
	- Download Sysmon: [HERE](https://docs.microsoft.com/en-us/sysinternals/downloads/sysmon)
	- Install: sysmon64  -i <configfile>
	- Update Configuration : sysmon64 -c <configfile>
	- Install event manifest: sysmon64 -m
	- Print schema: sysmon64 -s
	- Uninstall: sysmon64 -u [force]
4. Config file 
	This is a Microsoft Sysinternals Sysmon configuration file template with default high-quality event tracing. [HERE](https://github.com/SwiftOnSecurity/sysmon-config)

| Parameter | Description 
| --- | --- |
| -i | Install service and driver. Optionally take a configuration file |
| -c | Update configuration of an installed Sysmon driver or dump the current configuration if no other argument is provided. Optionally takes a configuration file |
| -m | Install the event manifest (implicitly done on service install as well) |
| -s | Print configuration schema definition |
| -u | Uninstall service and driver. Using -u force causes uninstall to proceed even when some components are not installed |




## Deploying Sysmon by GPO

- Login to server domain, create a folder containing file syscomon-controller.ps1
- Download file [Sysmon-controler.ps1](https://github.com/HASecuritySolutions/Sysmon-Manager/blob/main/sysmon_controller.ps1)
- Create GPO and link to the machines you want install
		
	•	Under Computer Configuration – Preferences – Windows Settings – Files, create a new file
![](../../images/sysmon/sysmon-1.png)

	•	Source file: \\ file path
	
	•	Destination file: C:\Windows\file sysmon
	![](../../images/sysmon/sysmon-2.png)
	
	•	Under Computer Configuration -> Preferences -> Control Panel Settings -> Scheduled Tasks, create a new Scheduled Task.
	
	![](../../images/sysmon/sysmon-3.png)
	
	•	Set the Action to Create. Set the name to Manage Sysmon (or whatever your preference is). Change the user to NT AUTHORITY\System. Set it to Run whether user is logged on or not. Then set Run with highest privileges.
	
	![](../../images/sysmon/sysmon-4.png)
	
	•	Switch to the Triggers tab. Click on New. Set the Settings to Daily and Time. Set repeat task to every 5 minutes or your preferred time frame. Also, set Stop all running tasks at end of repetition duration. Then click ok.
	
	![](../../images/sysmon/sysmon-5.png)
	
	•	Switch to the Actions tab. Click on New. Set program/script to powershell.exe. Then under Add arguments enter “-ExecutionPolicy Bypass -command "& C:\Windows\sysmon_controller.ps1”. Substitute your domain for yourdomain.int and do not include the double quotes. Click Ok
	
	![](../../images/sysmon/sysmon-6.png)
	
	•	Run cmd: gpupdate /force