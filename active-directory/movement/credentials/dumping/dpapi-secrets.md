# DPAPI Secrets

## <mark style="color:yellow;">ABOUT</mark>

<mark style="color:red;">**DPAPI Secrets**</mark> could be any data, as example browser creds, password manager data, vault creds.

<figure><img src="../../../../.gitbook/assets/image (2) (1) (1) (1).png" alt=""><figcaption><p>Image from Synacktiv article below</p></figcaption></figure>

## <mark style="color:yellow;">LINUX</mark>

> We'll be decrypting Secret with impacket's dpapi tool.

For decrypting DPAPI secret we need 3 things, and 2 of them are files which we need to install:

1. Masterkey (filename is guid)
2. DPAPI Secret
3. SID of user encrypted (directory where Secret located)
4. One of 5 things from scheme above, I'll use password for example.

### <mark style="color:blue;">Decrypting Secret</mark>

1. Decrypting Masterkey with password's hash and SID

```bash
impacket-dpapi masterkey -file 556a2412-1275-4ccf-b721-e6a0b4f90407 -password 'password123' -sid S-1-5-21-1487982659-1829050783-2281216199-1107
```

2. Using decrypted Masterkey to decrypt Secret

```bash
impacket-dpapi credential -f C8D69EBE9A43E9DEBF6B5FBD48B521B9 -key 0xd9a570722fbaf7149f9f9d691b0e137b7413c1414c452f9c77d6d8a8ed9efe3ecae990e047debe4ab8cc879e8ba99b31cdb7abad28408d8d9cbfdcaf319e9c84
```

## <mark style="color:yellow;">WINDOWS</mark>

> For Windows I only used SharpDPAPI for triaging certificates

### <mark style="color:blue;">SharpDPAPI</mark>

```powershell
.\SharpDPAPI.exe triage
```

## <mark style="color:yellow;">RESOURCES</mark>

{% embed url="https://www.thehacker.recipes/ad/movement/credentials/dumping/dpapi-protected-secrets#practice" %}

{% embed url="https://www.synacktiv.com/en/publications/windows-secrets-extraction-a-summary" %}

{% embed url="https://github.com/Flangvik/SharpCollection/blob/master/NetFramework_4.0_Any/SharpDPAPI.exe" %}
