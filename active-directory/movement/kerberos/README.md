---
icon: paw-simple
---

# Kerberos

## <mark style="color:$primary;">THEORY</mark>

{% embed url="https://venator17.gitbook.io/bibliotheque/active-directory/theory#kerberos" %}

## <mark style="color:$primary;">CLOCK ERROR</mark>

> `[-] Got error while trying to request TGT: Kerberos SessionError: KRB_AP_ERR_SKEW(Clock skew too great)`

It's very common (at least in HTB Boxes) error because of time zone difference timestamp header in authentication packets is getting to big. So the fix is time sync with server

#### Stop default automatic time sync

```bash
sudo systemctl stop systemd-timesyncd
```

#### Sync time with server

```bash
sudo ntpdate 13.13.13.13
```

#### Change time back

```bash
timedatectl set-ntp 1
```

## <mark style="color:$primary;">OTHER ERROR (CONFIGURATION FIX)</mark>

> I mainly noticed it when having authentication issues or some tooling requires it to properly interact with the machine.

```bash
netexec smb 13.13.13.13 -u 'songbird' -p 'p@ssw0rd' --generate-krb5-file ./krb5.conf
```
