# Port-Forwarding

## <mark style="color:$primary;">ABOUT</mark>

<mark style="color:red;">**Port forwarding**</mark> is a technique that allows us to <mark style="color:purple;">**redirect a communication request from one port to another**</mark>. Port forwarding uses **TCP** as the primary communication layer to provide interactive communication for the forwarded port. Basically if shortly:

* <mark style="color:yellow;">**Local Port-Forwarding**</mark> is like regular shell. Forwards our traffic through port to victim host port
* <mark style="color:yellow;">**Remote Port-Forwarding**</mark> is like reverse shell. To bypass firewall we make victim host to forward traffic to our host
* <mark style="color:yellow;">**Dynamic Port-Forwarding**</mark> is just a proxy, working with inbound and outbound traffic

## <mark style="color:$primary;">**LOCAL PORT FORWARDING**</mark>

Local port forwarding allows you to forward traffic from your local machine to a remote server. This is commonly used to access services behind a firewall or to create a secure channel for data transmission.

### <mark style="color:blue;">**Use case**</mark>

Accessing an intranet site or database from your local machine using SSH.

<figure><img src="../../.gitbook/assets/image (1) (2).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:blue;">**Example**</mark>

```bash
ssh -L <local-port>:<remote-ip>:<remote-port> user@<remote-ip>
```

## <mark style="color:$primary;">**REMOTE PORT FORWARDING**</mark>

Remote port forwarding allows a **remote system to open a listening port and forward incoming traffic through an SSH tunnel to a service running on your local machine or another internal host**.

### <mark style="color:blue;">**Use Case 1 — Accessing a Local Service from the Remote Host**</mark>

In this scenario, the remote system opens a port that <mark style="color:yellow;">**only the remote host itself can access**</mark> (typically bound to the loopback interface).

Traffic sent to this port is forwarded through the SSH tunnel to a **service running on the local machine**.

```bash
ssh -R <remote-port>:localhost:<local-app-port> user@<remote-machine>
```

<figure><img src="../../.gitbook/assets/Port-Forwarding_Socks5_Tunneling Schemes (1).png" alt=""><figcaption></figcaption></figure>

### <mark style="color:blue;">**Use Case 2 — Exposing a Service to the Remote Network**</mark>

In this scenario, the remote machine opens a port that is <mark style="color:yellow;">**accessible from other systems on the network**</mark>, not just the remote machine itself.

Traffic arriving at this port is forwarded through the SSH tunnel to a specified destination.

```bash
ssh -R <remote-bind-ip>:<remote-port>:<target-host>:<target-port> user@<remote-machine>
```

<figure><img src="../../.gitbook/assets/Port-Forwarding_Socks5_Tunneling Schemes(2).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:$primary;">**DYNAMIC PORT FORWARDING**</mark>

A SOCKS5 proxy created with SSH allows your local machine to <mark style="color:yellow;">**forward traffic from a local port to any host reachable from the remote machine**</mark>. Instead of forwarding traffic to a single fixed destination, <mark style="color:orange;">**it dynamically routes connections to different targets**</mark>.

This effectively allows your system to **use the remote machine as a network pivot**, accessing hosts that are only reachable from that machine. \
When combined with ProxyChains, many tools can transparently use this proxy for network scanning, enumeration, or accessing internal services.

### <mark style="color:blue;">**Use case**</mark>

Bypassing firewalls, anonymizing traffic, or routing web browsing through an SSH tunnel.

### <mark style="color:blue;">**Example**</mark>

```bash
ssh -D <local-port> user@<remote-ip>
```

This creates a SOCKS proxy on `localhost:<local-port>`. Applications configured to use this proxy will route traffic through the SSH server dynamically.

<figure><img src="../../.gitbook/assets/Port-Forwarding_Socks5_Tunneling Schemes(3).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:$primary;">RESOURCES</mark>

{% embed url="https://ittavern.com/visual-guide-to-ssh-tunneling-and-port-forwarding/" %}
