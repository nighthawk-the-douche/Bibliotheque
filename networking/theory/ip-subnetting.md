# IP / Subnetting

## <mark style="color:$primary;">IP</mark>

<mark style="color:red;">**IP or Internet Protocol is Network OSI layer**</mark> <mark style="color:purple;">**protocol**</mark>, which is used for identifying devices in the Internet. For this it uses IP addresses. Computer gets it's IP address from software and obtained automatically from a **DHCP** server.

## <mark style="color:$primary;">NIC</mark>

IP addresses, whether <mark style="color:purple;">**dynamic**</mark> or <mark style="color:purple;">**static**</mark>, are assigned to a <mark style="color:red;">**Network Interface Controller (NIC)**</mark>, also known as a <mark style="color:yellow;">**Network Adapter**</mark>. A system can have multiple **NICs** (both <mark style="color:purple;">**physical**</mark> and <mark style="color:purple;">**virtual**</mark>), allowing it to connect to different networks with multiple IP addresses.

## <mark style="color:$primary;">**Network Address**</mark>

<mark style="color:purple;">**Identifies a specific network and its range of IPs**</mark><mark style="color:purple;">**.**</mark>&#x20;

<mark style="color:yellow;">**Example:**</mark> <mark style="color:green;">**192.168.1.0/24**</mark> includes IPs from <mark style="color:green;">**192.168.1.1**</mark> to <mark style="color:green;">**192.168.1.254**</mark>.

## <mark style="color:$primary;">**Broadcast Address**</mark>

<mark style="color:purple;">**The highest IP in a subnet, used to send messages to all devices within the network.**</mark>

<mark style="color:yellow;">**Example:**</mark> In <mark style="color:green;">**192.168.1.0/24**</mark>, the broadcast address is <mark style="color:green;">**192.168.1.255**</mark>.

## <mark style="color:$primary;">**Gateway Address**</mark>

<mark style="color:purple;">**The router's IP that connects a network to others**</mark><mark style="color:purple;">**.**</mark>&#x20;

<mark style="color:yellow;">**Example:**</mark> <mark style="color:green;">**192.168.1.1**</mark> is the gateway for <mark style="color:green;">**192.168.1.0/24**</mark>.

## <mark style="color:$primary;">Loopback Address</mark>

<mark style="color:purple;">**A special IP used for a device to communicate with itself. Traffic never leaves the machine.**</mark>\
<mark style="color:yellow;">**Example:**</mark> <mark style="color:green;">**127.0.0.1**</mark> is the most common loopback address and is often called <mark style="color:red;">**localhost**</mark>.

## <mark style="color:$primary;">Wildcard Address</mark>

<mark style="color:purple;">**Represents all network interfaces on the current machine. Used when a service should listen on every available IP.**</mark>\
<mark style="color:yellow;">**Example:**</mark> If a machine has:

```bash
127.0.0.1    | (loopback)
192.168.1.10 | (LAN)
10.8.0.5     | (VPN)
```

A service bound to <mark style="color:green;">**0.0.0.0:80**</mark> will accept connections on:

```bash
127.0.0.1:80
192.168.1.10:80
10.8.0.5:80
```

## <mark style="color:$primary;">Routing Table</mark>

<mark style="color:red;">**A Router**</mark> is a networking <mark style="color:purple;">**device that forwards data packets between computer networks**</mark>. This device is usually connected to two or more different networks. When a packet arrives at a router, it examines destination IP address of the received packet and makes routing decisions accordingly. Routers use **Routing Tables** to determine which interface the packet will be sent out. <mark style="color:yellow;">**A routing table lists all networks for which routes are known.**</mark>

```bash
IPv4 Route Table                                                                                                                                                                              
================================================================================                                                                                                                   
Active Routes:                                                                                                                                                                                
Network Destination          Netmask          Gateway        Interface  Metric                                                                                                                   
            0.0.0.0          0.0.0.0        13.13.0.1      13.13.2.177      15                                                                                                                   
          13.13.0.0      255.255.0.0          On-link      13.13.2.177     271                                                                                                                   
        13.13.2.177  255.255.255.255          On-link      13.13.2.177     271                                                                                                                   
      13.13.255.255  255.255.255.255          On-link      13.13.2.177     271
          127.0.0.0        255.0.0.0          On-link        127.0.0.1     331
          127.0.0.1  255.255.255.255          On-link        127.0.0.1     331
    127.255.255.255  255.255.255.255          On-link        127.0.0.1     331
      192.168.100.0    255.255.255.0          On-link    192.168.100.1     271
      192.168.100.1  255.255.255.255          On-link    192.168.100.1     271
    192.168.100.255  255.255.255.255          On-link    192.168.100.1     271
          224.0.0.0        240.0.0.0          On-link        127.0.0.1     331
          224.0.0.0        240.0.0.0          On-link    192.168.100.1     271
          224.0.0.0        240.0.0.0          On-link      13.13.2.177     271
    255.255.255.255  255.255.255.255          On-link        127.0.0.1     331
    255.255.255.255  255.255.255.255          On-link    192.168.100.1     271
    255.255.255.255  255.255.255.255          On-link      13.13.2.177     271
===============================================================================
```

* <mark style="color:yellow;">**Network Destination:**</mark> \
  The target IP address or network segment the outbound packet is trying to reach.
* <mark style="color:yellow;">**Netmask:**</mark> \
  The subnet mask that dictates the size of the destination network and determines the most specific route match.
* <mark style="color:yellow;">**Gateway:**</mark> \
  The next-hop router IP address, or **"On-link"** if the destination is on the same local segment and requires no routing.
* <mark style="color:yellow;">**Interface:**</mark> \
  The local IP address of the logical adapter the operating system will use to push the traffic out.
* <mark style="color:yellow;">**Metric:**</mark> \
  The numerical cost assigned to the path, used by the OS to select the cheapest route when multiple options exist.

## <mark style="color:$primary;">**Network Interface (Logical)**</mark>

In a Windows routing table, the <mark style="color:red;">**Interface**</mark> column displays the <mark style="color:purple;">**IP address currently bound to a specific network driver**</mark>. It tells the operating system which logical exit point to use when pushing packets toward a destination.

The OS abstracts the underlying hardware. It does not care if the interface is:

* <mark style="color:yellow;">**Physical copper**</mark> Ethernet adapter.
* <mark style="color:yellow;">**Virtual switch**</mark> created by VMware, VirtualBox, or Hyper-V (which your `192.168.100.1` likely is).
* <mark style="color:yellow;">**VPN tunnel**</mark> (like a WireGuard `tun0` or OpenVPN `tap0` interface).
* <mark style="color:yellow;">**Software loopback**</mark> (`127.0.0.1`).

When the routing table instructs the system to use interface `13.13.2.177`, it simply means: "Hand the packet to the logical adapter that currently owns this IP address."

## <mark style="color:$primary;">Subnet</mark>

There are billions of devices, so to communicate fast and easy with each of it, IP connects firstly not to devices itself, but to <mark style="color:red;">**subnet**</mark>, in which this device is located. This is called <mark style="color:red;">**Scaling**</mark>. Also IP address is <mark style="color:yellow;">**32 bit number**</mark>, where every <mark style="color:yellow;">**8 bit**</mark> is called octet.&#x20;

<mark style="color:red;">**Subnet**</mark> - is the set of computers, which older part of IP address is same:

* <mark style="color:green;">**`312.245.10.1`**</mark>
* <mark style="color:green;">**`312.245.10.2`**</mark>
* <mark style="color:green;">**`312.245.10.3`**</mark>

### <mark style="color:$primary;">Subnet Mask</mark>

<mark style="color:red;">**Subnet mask**</mark> is <mark style="color:yellow;">**32-bit**</mark> <mark style="color:purple;">**number**</mark>, which shows us, where in IP address number of network, and where is host. Previously, **Subnet Classes** were used to classify subnets, but this scheme is outdated, so they started using <mark style="color:yellow;">**CIDR**</mark>, which means <mark style="color:yellow;">**Classless Inter-Domain Routing.**</mark>

**Mask structure:** bits with <mark style="color:red;">**1**</mark> is reserved for <mark style="color:red;">**network**</mark>, and cannot be changed. Otherwise bits with <mark style="color:blue;">**0**</mark> can be changed and reserved for <mark style="color:blue;">**hosts**</mark>. So the prefix <mark style="color:blue;">**/24**</mark>, <mark style="color:blue;">**/25**</mark>, <mark style="color:blue;">**/32**</mark> is just amount of reserved and non-touchable bits.

| Explanation                                          | Number                                                                                                                                                                                              |
| ---------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| <mark style="color:green;">**`IP(Decimal):`**</mark> | **213.180.193.3**                                                                                                                                                                                   |
| <mark style="color:green;">**`IP:`**</mark>          | <mark style="color:blue;">**11010101**</mark>**.**<mark style="color:blue;">**10110100**</mark>**.**<mark style="color:blue;">**11000001**</mark>**.**<mark style="color:blue;">**00000011**</mark> |
| <mark style="color:green;">**`Mask:`**</mark>        | <mark style="color:red;">**11111111**</mark>**.**<mark style="color:red;">**11111111**</mark>**.**<mark style="color:red;">**11111111**</mark>**.**<mark style="color:red;">**00000000**</mark>     |
| <mark style="color:green;">**`Subnet:`**</mark>      | <mark style="color:blue;">**11010101**</mark>**.**<mark style="color:blue;">**10110100**</mark>**.**<mark style="color:blue;">**11000001**</mark>**.**<mark style="color:red;">**00000000**</mark>  |

Subnet mask could be shown with 2 types: decimal and prefix

* <mark style="color:purple;">**Decimal**</mark>: <mark style="color:green;">**255.255.255.0**</mark>
* <mark style="color:purple;">**Prefix**</mark>: <mark style="color:green;">**/24**</mark>. Prefix 24 means 24 bits of subnet address in IP address.

The subnet mask does not have to end on an octet boundary

| Explanation                                              | Number                              |
| -------------------------------------------------------- | ----------------------------------- |
| <mark style="color:green;">**`IP(Decimal):`**</mark>     | 213.180.193.3 /20                   |
| <mark style="color:green;">**`IP:`**</mark>              | 11010101.10110100.11000001.00000011 |
| <mark style="color:green;">**`Mask:`**</mark>            | 11111111.11111111.11110000.00000000 |
| <mark style="color:green;">**`Subnet:`**</mark>          | 11010101.10110100.11000000.00000000 |
| <mark style="color:green;">**`Subnet(Decimal):`**</mark> | 213.180.192.0                       |
| <mark style="color:green;">**`Host(Decimal):`**</mark>   | 0.0.1.3                             |

So algorithm is that you should replace all **one's** in IP with **zero's** in mask at the same position.

## <mark style="color:$primary;">Manual Math</mark>

### <mark style="color:blue;">Binary to Decimal</mark>

<figure><img src="../../.gitbook/assets/image (8) (1).png" alt=""><figcaption><p>Made in Figma</p></figcaption></figure>

### <mark style="color:blue;">Decimal to Binary</mark>

<figure><img src="../../.gitbook/assets/image (7) (1) (1).png" alt=""><figcaption><p>Made in Figma</p></figcaption></figure>

### <mark style="color:blue;">Subnet hosts with Mask</mark>

All IP Address has <mark style="color:green;">**32**</mark> bits. <mark style="color:blue;">**/27**</mark> as example is amount of untouchable bits, so subnet mask would look like <mark style="color:blue;">**11111111.11111111.11111111.111**</mark><mark style="color:red;">**00000**</mark>. So the amount of <mark style="color:red;">**0's**</mark> is the bits we have for hosts <mark style="color:red;">**(5)**</mark>, and for calculating amount of accessible hosts we need only to do <mark style="color:orange;">**`2 ^ 5 = 32`**</mark>. Which means we now have <mark style="color:green;">**32**</mark> accessible hosts with this subnet mask

### <mark style="color:blue;">Dividing into subnets</mark>

As example we would divide <mark style="color:green;">**10.200.20.0/27**</mark> network into **4** different subnets.

**Divide the network into 4 subnets:**

* <mark style="color:orange;">**`32 / 4 = 8`**</mark>, so each subnet will contain <mark style="color:yellow;">**8 IP addresses**</mark>, including <mark style="color:blue;">**network**</mark> and <mark style="color:red;">**broadcast**</mark> addresses.
*   **Calculate the ranges for each subnet:**

    * Start with the **network address (10.200.20.0)** and increment by 8 for each new subnet.
    * For each range:
      * The **first IP** is the <mark style="color:blue;">**network address**</mark> of the subnet.
      * The **last IP** is the <mark style="color:red;">**broadcast address**</mark> of the subnet.

    Subnet calculations:

    * **Subnet 1:**
      * <mark style="color:green;">**Range:**</mark> **10.200.20.0 - 10.200.20.7**
      * <mark style="color:blue;">**Network address:**</mark> **10.200.20.0**
      * <mark style="color:red;">**Broadcast address:**</mark> **10.200.20.7**
    * **Subnet 2:**
      * <mark style="color:green;">**Range:**</mark> **10.200.20.8 - 10.200.20.15**
      * <mark style="color:blue;">**Network address:**</mark> **10.200.20.8**
      * <mark style="color:red;">**Broadcast address:**</mark> **10.200.20.15**
    * **Subnet 3:**
      * <mark style="color:green;">**Range:**</mark> **10.200.20.16 - 10.200.20.23**
      * <mark style="color:blue;">**Network address:**</mark> **10.200.20.16**
      * <mark style="color:red;">**Broadcast address:**</mark> **10.200.20.23**
    * **Subnet 4:**
      * <mark style="color:green;">**Range:**</mark> **10.200.20.24 - 10.200.20.31**
      * <mark style="color:blue;">**Network address:**</mark> **10.200.20.24**
      * <mark style="color:red;">**Broadcast address:**</mark> **10.200.20.31**

## <mark style="color:$primary;">RESOURCES</mark>

{% embed url="https://www.geeksforgeeks.org/computer-networks/routing-tables-in-computer-network/" %}
