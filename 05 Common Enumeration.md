# Common Enumeration
## Basic Info
- Windows 10 Pro 10586 x64
- Latest patch for this OS build was April 2018
![[Pasted image 20220802122633.png]]

## Nmap
```
PORT      STATE SERVICE      VERSION
80/tcp    open  http         Microsoft IIS httpd 10.0
|_http-title: Ask Jeeves
| http-methods:
|   Supported Methods: OPTIONS TRACE GET HEAD POST
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
135/tcp   open  msrpc        Microsoft Windows RPC
445/tcp   open  microsoft-ds Microsoft Windows 7 - 10 microsoft-ds (workgroup: WORKGROUP)
50000/tcp open  http         Jetty 9.4.z-SNAPSHOT
|_http-title: Error 404 Not Found
|_http-server-header: Jetty(9.4.z-SNAPSHOT)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Microsoft Windows Server 2008 R2 (91%), Microsoft Windows 10 1511 - 1607 (87%), Microsoft Windows 8.1 Update 1 (86%), Microsoft Windows Phone 7.5 or 8.0 (86%), FreeBSD 6.2-RELEASE (86%), Microsoft Windows 10 1607 (85%), Microsoft Windows 10 1511 (85%), Microsoft Windows 7 or Windows Server 2008 R2 (85%), Microsoft Windows Server 2008 R2 or Windows 8.1 (85%), Microsoft Windows Server 2008 R2 SP1 or Windows 8 (85%)
No exact OS matches for host (test conditions non-ideal).
Uptime guess: 0.004 days (since Tue Aug  2 12:14:58 2022)
Network Distance: 2 hops
TCP Sequence Prediction: Difficulty=261 (Good luck!)
IP ID Sequence Generation: Incremental
Service Info: Host: JEEVES; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time:
|   date: 2022-08-02T16:19:26
|_  start_date: 2022-08-02T16:15:09
| smb2-security-mode:
|   3.1.1:
|_    Message signing enabled but not required
| smb-security-mode:
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_clock-skew: mean: 4h59m57s, deviation: 0s, median: 4h59m57s

TRACEROUTE (using port 80/tcp)
HOP RTT      ADDRESS
1   29.43 ms 10.10.14.1
2   32.46 ms 10.10.10.63

NSE: Script Post-scanning.
Initiating NSE at 12:20
Completed NSE at 12:20, 0.00s elapsed
Initiating NSE at 12:20
Completed NSE at 12:20, 0.00s elapsed
Initiating NSE at 12:20
Completed NSE at 12:20, 0.00s elapsed
Read data files from: /usr/bin/../share/nmap
OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 170.61 seconds
           Raw packets sent: 131244 (5.778MB) | Rcvd: 152 (11.224KB)

```

## HTTP - 80

![[Pasted image 20220802121647.png]]

- On submitting 'hello'
![[Pasted image 20220802121944.png]]

![[Pasted image 20220802121959.png]]

- Oddly the error message appears to be a picture, page source
```
<img src="jeeves.PNG" width="90%" height="100%">
```

- Version information does not line up with the fact that CME thinks its windows 10 x64?

- Background picture has a strange name? `Ask-Jeeves-whatever-happened-to-32225327-270-301`
![[Pasted image 20220802124910.png]]

- Tried big.txt and directory-list-2.3-medium with various extensions and found nothing
## HTTP - 50000
- An instance of Jetty 
- Tried some of the exploits, got nothing
- Dirbusting with big.txt found nothing but directory-list-2.3-medium does find `/askjeeves`
- Find an unauthenticated jenkins instance
![[Pasted image 20220802132425.png]]

## RPC
- No guest or null session
```
┌──(kali㉿kali)-[~/oscp/boxes/htb/jeeves]
└─$ rpcclient -U 'guest' 10.10.10.63                                                                                                                                                     1 ⨯
Password for [WORKGROUP\guest]:
Cannot connect to server.  Error was NT_STATUS_ACCOUNT_DISABLED

┌──(kali㉿kali)-[~/oscp/boxes/htb/jeeves]
└─$ rpcclient -U '%' 10.10.10.63                                                                                                                                                         1 ⨯
Cannot connect to server.  Error was NT_STATUS_ACCESS_DENIED

┌──(kali㉿kali)-[~/oscp/boxes/htb/jeeves]
└─$ rpcclient -U 'guest%' 10.10.10.63                                                                                                                                                    1 ⨯
Cannot connect to server.  Error was NT_STATUS_ACCOUNT_DISABLED

┌──(kali㉿kali)-[~/oscp/boxes/htb/jeeves]
└─$ rpcclient -U 'jeeves%' 10.10.10.63                                                                                                                                                   1 ⨯
Cannot connect to server.  Error was NT_STATUS_LOGON_FAILURE

```

## SMB
- CME
```
┌──(kali㉿kali)-[~/oscp/boxes/htb/jeeves]
└─$ crackmapexec smb 10.10.10.63                                                                                                                                                         1 ⨯
SMB         10.10.10.63     445    JEEVES           [*] Windows 10 Pro 10586 x64 (name:JEEVES) (domain:Jeeves) (signing:False) (SMBv1:True)

```

- No null or guest sessions either
```
┌──(kali㉿kali)-[~/oscp/boxes/htb/jeeves]
└─$ crackmapexec smb 10.10.10.63 -u '' -p ''
SMB         10.10.10.63     445    JEEVES           [*] Windows 10 Pro 10586 x64 (name:JEEVES) (domain:Jeeves) (signing:False) (SMBv1:True)
SMB         10.10.10.63     445    JEEVES           [-] Jeeves\: STATUS_ACCESS_DENIED                             

┌──(kali㉿kali)-[~/oscp/boxes/htb/jeeves]
└─$ crackmapexec smb 10.10.10.63 -u 'geust' -p ''
SMB         10.10.10.63     445    JEEVES           [*] Windows 10 Pro 10586 x64 (name:JEEVES) (domain:Jeeves) (signing:False) (SMBv1:True)
SMB         10.10.10.63     445    JEEVES           [-] Jeeves\geust: STATUS_LOGON_FAILURE                                                                                                                                     
┌──(kali㉿kali)-[~/oscp/boxes/htb/jeeves]
└─$ crackmapexec smb 10.10.10.63 -u 'guest' -p ''
SMB         10.10.10.63     445    JEEVES           [*] Windows 10 Pro 10586 x64 (name:JEEVES) (domain:Jeeves) (signing:False) (SMBv1:True)
SMB         10.10.10.63     445    JEEVES           [-] Jeeves\guest: STATUS_ACCOUNT_DISABLED

┌──(kali㉿kali)-[~/oscp/boxes/htb/jeeves]
└─$ crackmapexec smb 10.10.10.63 -u '' -p '' --shares
SMB         10.10.10.63     445    JEEVES           [*] Windows 10 Pro 10586 x64 (name:JEEVES) (domain:Jeeves) (signing:False) (SMBv1:True)
SMB         10.10.10.63     445    JEEVES           [-] Jeeves\: STATUS_ACCESS_DENIED
SMB         10.10.10.63     445    JEEVES           [-] Error enumerating shares: Error occurs while reading from remote(104)

```

## HTTP - 50000
![[Pasted image 20220802123003.png]]

- According to the eclipse website
```
# The Eclipse Jetty Project

Jetty provides a web server and servlet container, additionally providing support for HTTP/2, WebSocket, OSGi, JMX, JNDI, JAAS and many other integrations. These components are open source and are freely available for commercial use and distribution.

Jetty is used in a wide variety of projects and products, both in development and production. Jetty has long been loved by developers due to its long history of being easily embedded in devices, tools, frameworks, application servers, and modern cloud services.

 

Features

Jetty Powered

-   Full-featured and standards-based
    
-   Open source and commercially usable
    
-   Flexible and extensible
    
-   Small footprint
    
-   Embeddable
    
-   Asynchronous
    
-   Enterprise scalable
    
-   Dual licensed under Apache and Eclipse
    

-   Large clusters, such as Facebook Presto
    
-   Cloud computing, such as Google AppEngine
    
-   SaaS, such as Yahoo! and Zimbra
    
-   Application Servers, such as Apache Geronimo
    
-   Frameworks, such as GWT
    
-   Tools, such as the Eclipse IDE
    
-   Devices, such as Android
    
-   More…​
```

```
┌──(kali㉿kali)-[~/oscp/boxes/htb/jeeves]
└─$ searchsploit jetty
----------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                                                             |  Path
----------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Eclipse Jetty 11.0.5 - Sensitive File Disclosure                                                                                                           | java/webapps/50478.txt
Jetty 3.1.6/3.1.7/4.1 Servlet Engine - Arbitrary Command Execution                                                                                         | cgi/webapps/21895.txt
Jetty 4.1 Servlet Engine - Cross-Site Scripting                                                                                                            | jsp/webapps/21875.txt
Jetty 6.1.x - JSP Snoop Page Multiple Cross-Site Scripting Vulnerabilities                                                                                 | jsp/webapps/33564.txt
jetty 6.x < 7.x - Cross-Site Scripting / Information Disclosure / Injection                                                                                | jsp/webapps/9887.txt
Jetty 9.4.37.v20210219 - Information Disclosure                                                                                                            | java/webapps/50438.txt
Jetty Web Server - Directory Traversal                                                                                                                     | windows/remote/36318.txt
Mortbay Jetty 7.0.0-pre5 Dispatcher Servlet - Denial of Service                                                                                            | multiple/dos/8646.php
----------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
Papers: No Results

```