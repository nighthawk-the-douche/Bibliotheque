# IP / Subnetting

## <mark style="color:yellow;">IP</mark>

<mark style="color:red;">**IP or Internet Protocol is Network OSI layer**</mark> <mark style="color:purple;">**protocol**</mark>, which is used for identifying devices in the Internet. For this it uses IP addresses. Computer gets it's IP address from software and obtained automatically from a **DHCP** server.

## <mark style="color:yellow;">NIC</mark>

IP addresses, whether <mark style="color:purple;">**dynamic**</mark> or <mark style="color:purple;">**static**</mark>, are assigned to a <mark style="color:red;">**Network Interface Controller (NIC)**</mark>, also known as a <mark style="color:yellow;">**Network Adapter**</mark>. A system can have multiple **NICs** (both <mark style="color:purple;">**physical**</mark> and <mark style="color:purple;">**virtual**</mark>), allowing it to connect to different networks with multiple IP addresses.

## <mark style="color:yellow;">**Network Address**</mark>

<mark style="color:purple;">**Identifies a specific network and its range of IPs**</mark>.&#x20;

<mark style="color:yellow;">**Example:**</mark> <mark style="color:green;">**192.168.1.0/24**</mark> includes IPs from <mark style="color:green;">**192.168.1.1**</mark> to <mark style="color:green;">**192.168.1.254**</mark>.

## <mark style="color:yellow;">**Broadcast Address**</mark>

<mark style="color:purple;">**The highest IP in a subnet, used to send messages to all devices within the network.**</mark>

<mark style="color:yellow;">**Example:**</mark> In <mark style="color:green;">**192.168.1.0/24**</mark>, the broadcast address is <mark style="color:green;">**192.168.1.255**</mark>.

## <mark style="color:yellow;">**Gateway Address**</mark>

<mark style="color:purple;">**The router's IP that connects a network to others**</mark>.&#x20;

<mark style="color:yellow;">**Example:**</mark> <mark style="color:green;">**192.168.1.1**</mark> is the gateway for <mark style="color:green;">**192.168.1.0/24**</mark>.

## <mark style="color:yellow;">Subnet</mark>

There are billions of devices, so to communicate fast and easy with each of it, IP connects firstly not to devices itself, but to <mark style="color:red;">**subnet**</mark>, in which this device is located. This is called <mark style="color:red;">**Scaling**</mark>. Also IP address is <mark style="color:yellow;">**32 bit number**</mark>, where every <mark style="color:yellow;">**8 bit**</mark> is called octet.&#x20;

<mark style="color:red;">**Subnet**</mark> - is the set of computers, which older part of IP address is same:

* <mark style="color:green;">**`312.245.10.1`**</mark>
* <mark style="color:green;">**`312.245.10.2`**</mark>
* <mark style="color:green;">**`312.245.10.3`**</mark>

### <mark style="color:yellow;">Subnet Mask</mark>

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

## <mark style="color:yellow;">Manual Math</mark>

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
