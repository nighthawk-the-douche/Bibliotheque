---
icon: eye
---

# Nmap

## <mark style="color:$primary;">ABOUT</mark>

<mark style="color:red;">**Nmap**</mark> is a <mark style="color:purple;">**complex port-scanning tool**</mark> which is very often used by security specialists. It's more complex that **ncat** so in some ways sysadmins or programmers could use them to check hosts and port available.

<figure><img src="../.gitbook/assets/photo_2024-10-22_15-37-44.jpg" alt=""><figcaption><p>Basic nmap version+script scanning</p></figcaption></figure>

And it's a lot more complex than other tool ncat, which we can use for port-scanning purposes too

<figure><img src="../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption><p>basic netcat port check/scan</p></figcaption></figure>

## <mark style="color:$primary;">CheatSheet</mark>

| Option                                                 | Description                                                         |
| ------------------------------------------------------ | ------------------------------------------------------------------- |
| <mark style="color:green;">`13.13.13.0/24`</mark>      | Target network range.                                               |
| <mark style="color:green;">`-sn`</mark>                | Disables port scanning.                                             |
| <mark style="color:green;">`-Pn`</mark>                | Disables ICMP Echo Requests.                                        |
| <mark style="color:green;">`-n`</mark>                 | Disables DNS Resolution.                                            |
| <mark style="color:green;">`-PE`</mark>                | Performs the ping scan using ICMP Echo Requests against the target. |
| <mark style="color:green;">`--packet-trace`</mark>     | Shows all packets sent and received.                                |
| <mark style="color:green;">`--reason`</mark>           | Displays the reason for a specific result.                          |
| <mark style="color:green;">`--disable-arp-ping`</mark> | Disables ARP Ping Requests.                                         |
| <mark style="color:green;">`--top-ports=<num>`</mark>  | Scans the specified top ports defined as most frequent.             |
| <mark style="color:green;">`-p-`</mark>                | Scan all ports.                                                     |
| <mark style="color:green;">`-p22-110`</mark>           | Scan all ports between 22 and 110.                                  |
| <mark style="color:green;">`-p22,25`</mark>            | Scans only the specified ports 22 and 25.                           |
| <mark style="color:green;">`-F`</mark>                 | Scans top 100 ports.                                                |
| <mark style="color:green;">`-sS`</mark>                | Performs a TCP SYN-Scan.                                            |
| <mark style="color:green;">`-sA`</mark>                | Performs a TCP ACK-Scan.                                            |
| <mark style="color:green;">`-sU`</mark>                | Performs a UDP Scan.                                                |
| <mark style="color:green;">`-sV`</mark>                | Scans the discovered services for their versions.                   |
| <mark style="color:green;">`-sC`</mark>                | Perform a Script Scan with scripts categorized as "default".        |
| <mark style="color:green;">`--script=<script>`</mark>  | Performs a Script Scan using the specified scripts.                 |
| <mark style="color:green;">`-O`</mark>                 | Performs an OS Detection Scan to determine the OS of the target.    |
| <mark style="color:green;">`-A`</mark>                 | Performs OS Detection, Service Detection, and traceroute scans.     |
| <mark style="color:green;">`-D RND:5`</mark>           | Sets the number of random Decoys used to scan the target.           |
| <mark style="color:green;">`-e`</mark>                 | Specifies the network interface used for the scan.                  |
| <mark style="color:green;">`-S 13.13.13.200`</mark>    | Specifies the source IP address for the scan.                       |
| <mark style="color:green;">`-g`</mark>                 | Specifies the source port for the scan.                             |
| <mark style="color:green;">`--dns-server <ns>`</mark>  | DNS resolution is performed using a specified name server.          |

### <mark style="color:blue;">Output Options</mark>

| Option                                           | Description                                                                    |
| ------------------------------------------------ | ------------------------------------------------------------------------------ |
| <mark style="color:green;">`-oA filename`</mark> | Stores the results in all available formats starting with the name "filename". |
| <mark style="color:green;">`-oN filename`</mark> | Stores the results in normal format with the name "filename".                  |
| <mark style="color:green;">`-oG filename`</mark> | Stores the results in "grepable" format with the name "filename".              |
| <mark style="color:green;">`-oX filename`</mark> | Stores the results in XML format with the name "filename".                     |

### <mark style="color:blue;">Performance Options</mark>

| Option                                                         | Description                                                  |
| -------------------------------------------------------------- | ------------------------------------------------------------ |
| <mark style="color:green;">`--max-retries <num>`</mark>        | Sets the number of retries for scans of specific ports.      |
| <mark style="color:green;">`--stats-every=5s`</mark>           | Displays the scan's status every 5 seconds.                  |
| <mark style="color:green;">`-v/-vv`</mark>                     | Displays verbose output during the scan.                     |
| <mark style="color:green;">`--initial-rtt-timeout 50ms`</mark> | Sets the specified time value as the initial RTT timeout.    |
| <mark style="color:green;">`--max-rtt-timeout 100ms`</mark>    | Sets the specified time value as the maximum RTT timeout.    |
| <mark style="color:green;">`--min-rate 300`</mark>             | Sets the number of packets that will be sent simultaneously. |
| <mark style="color:green;">`-T <0-5>`</mark>                   | Specifies the specific timing template.                      |

## <mark style="color:$primary;">Templates</mark>

#### <mark style="color:$success;">**Full Intense Scan (Aggressive & Detailed)**</mark>

```powershell
nmap -A -T4 -p- -sS -sV -O 13.13.13.13
```

#### <mark style="color:$success;">Fast, Stealthy Scan (Avoid Detection)</mark>

```powershell
nmap -sS -T3 -Pn -f --randomize-hosts --data-length 100 -D RND:10 13.13.13.13
```

#### <mark style="color:$success;">**Scan Multiple Targets**</mark>

```powershell
nmap -sS -sV -p 21,22,80,443 -T4 -iL targets.txt
```

#### <mark style="color:$success;">Quick Internal Scan</mark>

```bash
nmap -sn 13.13.13.0/24
```

#### <mark style="color:$success;">SYN, Detection Network</mark>

```bash
nmap -sS -sV -O -p- 13.13.13.0/24
```

## <mark style="color:$primary;">NetScanner</mark>

This is my python script which uses nmap for 4 phases to effectively use time with logging. (Maybe I'll make the same thing for rustscan to have speed too)

{% embed url="https://github.com/venator17/NetScanner" %}

