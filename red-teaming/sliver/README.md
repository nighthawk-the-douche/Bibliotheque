---
icon: snake
---

# Sliver

## <mark style="color:$primary;">ABOUT</mark>

Sliver is a powerful command and control (C2) framework designed to provide advanced capabilities for covertly managing and controlling remote systems. With Sliver, security professionals, red teams, and penetration testers can easily establish a secure and reliable communication channel over Mutual TLS, HTTP(S), DNS, or Wireguard with target machines. Enabling them to execute commands, gather information, and perform various post-exploitation activities. The framework offers a user-friendly console interface, extensive functionality, and support for multiple operating systems as well as multiple CPU architectures, making it an indispensable tool for conducting comprehensive offensive security operations.

## <mark style="color:$primary;">ARCHITECTURE</mark>

```
             In         ┌───────────────┐ C2
┌─────────┐  Memory     │               │ Protocol ┌─────────┐
│ Server  ├────────────►│ Sliver Server ├─────────►│ Implant │
│ Console │             │               │          └─────────┘
└─────────┘             └───────────────┘
                               ▲
                               │
                               │gRPC/mTLS
                               │
                          ┌────┴────┐
                          │ Sliver  │
                          │ Client  │
                          └─────────┘
```

<figure><img src="../../.gitbook/assets/image (15).png" alt=""><figcaption></figcaption></figure>

#### <mark style="color:$success;">Server</mark>

Sliver Server is also part of the `sliver-server` executable and manages the internal database, starts/stops network listeners (such as C2 listeners, though there are other types). The main interface used to interact with the server is the gRPC interface, thru which all functionality is implemented. By default the server will only start an in-memory gRPC listener that can only be communicated with from the server console. However, the gRPC interface can also be exposed to the network (i.e., multiplayer mode) over mutual TLS (mTLS).

#### <mark style="color:$success;">Client</mark>

Sliver Client component has the role of being the location the user will execute the commands and tools needed to fulfill their objectives. It's a necessity if operators wants to use same server, then they need to install client.

{% hint style="info" %}
Both Server and Client has their own consoles, but server console is just upgraded version of client console, except server-only features. So the syntax is mostly the same.
{% endhint %}

#### <mark style="color:$success;">Implant</mark>

[**\[DOCS\]**](https://sliver.sh/docs?name=Getting%20Started#implants-beacon-vs-session)

The implant is the actual malicious code run on the target system you want remote access to. And sliver is stage 2 payload, so because of easy static compilation in Go file can be relatively large. That's why it's better to use stagers (when file size is a concern).

Sliver implants support two modes of operation: <mark style="color:$success;">**"beacon mode"**</mark> and <mark style="color:$success;">**"session mode"**</mark>. Beacon mode implements an asynchronous communication style where the implant periodically checks in with the server, retrieves tasks, executes them, and returns the results. In "session mode" the implant will create an interactive real-time session using either a persistent connection or long polling depending on the underlying C2 protocol.

{% hint style="info" %}
Beacons may be tasked to open interactive sessions over _any C2 protocol they were compiled with_ using the `interactive` command, i.e., if a beacon implant was not compiled with HTTP C2 it cannot open a session over HTTP (use the `close` command to close the session). Currently implants initially compiled for session mode cannot be converted to beacon mode.
{% endhint %}

#### <mark style="color:$success;">Stager</mark>

[**\[DOCS\]**](https://sliver.sh/docs?name=Stagers)

As payloads can be pretty big (around 10MB), you may sometime require the use of stagers to execute your implant on a target system. Stager downloads implant shellcode from a remote location, such as the C2 server, and then runs the shellcode.

#### <mark style="color:$success;">Armory</mark>

Its capability of having pre-installed .NET binaries ready to be used makes the operators' lives easier. However, one of the drawbacks that one might stumble upon is the detection of the tools. In the future, we may need to think of a way to change the internals of the tools to avoid being detected.

## <mark style="color:$primary;">OPERATORS</mark>

{% hint style="info" %}
For both components of `Sliver`, we can use the `help` command followed by any available command in the tool to get more detailed information, such as brief information about the command, its arguments, etc.

```bash
[server] sliver > help new-operator
```
{% endhint %}

### <mark style="color:yellow;">Adding an operator</mark>

Sliver can differentiate who can connect based on the generated profile from its server. `-l` parameter is server's IP so that operator can connect to it.

```bash
[server] sliver > new-operator -n songbird -l 13.13.13.13
```

{% hint style="warning" %}
Before clients can connect to a server you must start an RPC listener with the `multiplayer`.The default port is TCP/31337.
{% endhint %}

### <mark style="color:yellow;">Setting up a client</mark>

```bash
./sliver-client_linux import songbird_13.13.13.13.cfg
```

## <mark style="color:$primary;">INSTALLATION</mark>

You can use either Github Releases [**\[LINK\]**](https://github.com/BishopFox/sliver/releases) or&#x20;

<details>

<summary>Github Releases</summary>

[https://github.com/BishopFox/sliver/releases](https://github.com/BishopFox/sliver/releases)

```bash
wget -q https://github.com/BishopFox/sliver/releases/download/{your-version}/sliver-server_linux
chmod +x ./sliver-server_linux
```

</details>

<details>

<summary>Linux One-liner</summary>

```bash
curl https://sliver.sh/install|sudo bash
```

{% hint style="info" %}
If you install Sliver via the one-liner, you can check that the server service is running using `systemctl status sliver`. Note that the Sliver service is not configured to start automatically on boot by default (i.e., if you reboot the server you'll need to start the service again using `systemctl start sliver`):
{% endhint %}

</details>

## <mark style="color:$primary;">RESOURCES</mark>

{% embed url="https://sliver.sh/" %}

{% embed url="https://sliver.sh/tutorials?name=1+-+Getting+Started" %}

{% embed url="https://sliver.sh/docs" %}

{% embed url="https://seriotonctf.github.io/2024/06/30/Getting-started-with-Sliver-C2/" %}

{% embed url="https://dominicbreuker.com/post/" %}
