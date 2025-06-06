# 42-netpractice
This project is a simulation that challenges us to troubleshoot computer networks by configuring routers and hosts using IP addressing and routing principles. The goal is to understand how data travels through networks and how to ensure proper communication between different subnets and hosts.

## Key concepts
### Network
A **network** is a group of two or more devices (like computers, smartphones, servers, or routers) that are connected and able to communicate with each other to share data and resources.

The **Internet** is the largest network in the world. It's actually a network of networks, meaning it's made up of thousands of smaller networks all connected together.<br>
Each time you an e-mail is sent or a website is opened, data is sent through many routers and networks, all cooperating to deliver the request. The data hops between different routers and IP addresses, following paths based on routing rules, until it reaches its destination.


### IP Address & Subnet Mask
An **IP address** (Internet Protocol address) uniquely identifies a device on a network. It consists of two parts:
* The **network portion** identifies the specific network the device belongs to
* The **host portion** identifies a unique device within that network

To distinguish these portions, a **subnet mask** is used. The subnet mask uses a contiguous block of `1`s (from the left) to represent the network portion, followed by `0`s for the host portion.

Crucially, **devices must have the same network portion** in their IP addresses (as defined by the subnet mask) to communicate directly within the same local network.

For this project, we focus on IPv4, where both the IP address and the subnet mask are divided into four 8-bit blocks (called octets), each ranging from 0 to 255. <br><br>

**CIDR (Classless Inter-Domain Routing)** notation is a compact way to represent an IP address and its associated subnet mask. It appends a slash (`/`) followed by a number indicating how many bits (from the left) belong to the network portion.<br>
The table below shows common CIDR prefixes, their corresponding subnet masks, and the implications for network size:

| **CIDR** | **Subnet Mask** | **# Network Bits** | **# Host Bits** | **# Hosts per Subnet** |
| -------- | --------------- | ------------------ | --------------- | ---------------------- |
| /8       | 255.0.0.0       | 8                  | 24              | 16,777,214             |
| /12      | 255.240.0.0     | 12                 | 20              | 1,048,574              |
| /16      | 255.255.0.0     | 16                 | 16              | 65,534                 |
| /24      | 255.255.255.0   | 24                 | 8               | 254                    |
| /25      | 255.255.255.128 | 25                 | 7               | 126                    |
| /30      | 255.255.255.252 | 30                 | 2               | 2                      |

*Note on Usable Hosts:* if `n` is the number of host bits, a subnet can theoretically have 2<sup>n</sup> addresses. However, two addresses are always reserved:
1.  The **Network Address**: the first address in the range (all host bits are 0) - it identifies the network itself
2.  The **Broadcast Address**: the last address in the range (all host bits are 1) - packets sent to this address are delivered to all hosts on that network

Therefore, the number of usable IP addresses for hosts is 2<sup>n</sup> - 2.<br><br>

**Private IP** addresses are used inside local networks (LANs) and are not directly routable from the public Internet. They are defined in these ranges, as per RFC 1918:
| **Class**        | **Private IP Range**          | **CIDR Notation**        | **Notes**                     |
|------------------|-------------------------------|--------------------------|-------------------------------|
| A                | 10.0.0.0 - 10.255.255.255     | 10.0.0.0/8               | Large private networks        |
| B                | 172.16.0.0 - 172.31.255.255   | 172.16.0.0/12            | Medium-sized private networks |
| C                | 192.168.0.0 - 192.168.255.255 | 192.168.0.0/16           | Home/small private networks   |


### Routing
Routers use **routing tables** to decide where to forward data packets. Each entry in the table defines:
*  A **destination network** address and its subnet mask
*  The **next hop** IP address (usually another router's IP address or an outgoing interface) to reach that network

When a router receives a packet, it examines the destination IP address and compares it against the entries in its routing table. It chooses the most specific matching route (the one with the longest prefix match) and forwards the packet accordingly.<br>
If no exact match is found, the router may use a **default route** (`0.0.0.0/0`) to send the packet toward a general gateway. If no default route is set, the router will discard the packet.

Routers allow devices from different networks (with different IP ranges) to communicate by forwarding packets between them, applying appropriate routing logic and sometimes performing Network Address Translation (NAT) when crossing into public networks like the Internet.


## Network simulations
<details>
  <summary><strong>Level 1</strong></summary>
  
  * Host A needs to communicate with host B
  * Host C needs to communicate with host D
  <p align="center">
    <img src="img/level1.png"><br />
  </p>
  
  To allow direct communication between devices, their IP addresses must belong to the same subnet.
  
  Subnets:
  * Host A and host B (`/24`): `104.96.23.0` - `104.96.23.255`
  * Host C and host D (`/16`): `211.191.0.1` - `211.191.255.255`
  
  > *Note: the first (network) and last (broadcast) IP in each range cannot be assigned to devices*
  <br>
</details>

<details>
  <summary><strong>Level 2</strong></summary>
  
  * Host A needs to communicate with host B
  * Host C needs to communicate with host D
  <p align="center">
    <img src="img/level2.png"><br />
  </p>
  
  To allow direct communication between devices, their IP addresses must belong to the same subnet.

  Subnets:
  * Host A and host B (`/27`): `192.168.91.192` - `192.168.91.223`
  * Host C and host D (`/30`): `42.42.42.40` - `42.42.42.43`

  > *You can use any range of public IP addresses, as long as they are not reserved (private IPs)*
  <br>
</details>

<details>
  <summary><strong>Level 3</strong></summary>

  * Host A needs to communicate with host B
  * Host A needs to communicate with host C
  * Host B needs to communicate with host C
  <p align="center">
    <img src="img/level3.png"><br />
  </p>

  All three hosts are connected to the same switch, so they must belong to the same subnet to communicate directly.

  Subnet:
  * Host A, host B and host C (`/25`): `104.198.6.0` - `104.198.6.127`
  <br>
</details>

<details>
  <summary><strong>Level 4</strong></summary>

  * Host A needs to communicate with host B
  * Host A needs to communicate with router R
  * Host B needs to communicate with router R
  <p align="center">
    <img src="img/level4.png"><br />
  </p>

  Router R has three interfaces:
  * Interface R2 (`/25`): `60.40.117.0` - `60.40.117.127`
  * Interface R3 (`/26`): `60.40.117.192` - `60.40.117.255`

  For the subnet between interface R1, host A and host B (all connected to the same switch), we need to choose a subnet that:
  * Includes host A’s IP address (`60.40.117.132`)
  * Does not overlap with the existing subnets used by R2 and R3

  Subnet:
  * R1, Host A, and Host B (`/28`): `60.40.117.128` – `60.40.117.143`
  <br>
</details>

<details>
  <summary><strong>Level 5</strong></summary>

  * Host A needs to communicate with host B
  * Host A needs to communicate with router R
  * Host B needs to communicate with router R
  <p align="center">
    <img src="img/level5.png"><br />
  </p>

  Subnets:
  * Host A and interface R1 (`/25`): `58.232.46.0` - `58.232.46.127`
  * Host B and interface R2 (`/18`): `168.70.128.0` - `168.70.191.255`
  > *Hosts A and B are on different IP subnets and cannot communicate directly*

  Routing:
  * Both host A and host B are configured to use router R as their default gateway
  * Router R has two interfaces (R1 and R2), one in each subnet, and is responsible for forwarding packets between Host A and Host B
  * This allows inter-subnet communication, as the router routes traffic from one subnet to the other using standard IP forwarding
  <br>
</details>

<details>
  <summary><strong>Level 6</strong></summary>

  * Host A needs to communicate with interface Somewhere on the net
  <p align="center">
    <img src="img/level6.png"><br />
  </p>

  Subnets:
  * Host A and interface R1 (`/25`): `68.180.143.128` - `68.180.143.255`
  * Interface R2 (`/28`): `163.172.250.0` - `163.172.250.15`

  Routing:
  * Host A uses the local router (interface R1) as its default gateway
  * The router forwards packets to the Internet via its R2 interface
  * Router R’s default route points to `163.172.250.1`, enabling access to external networks
  * The Internet forwards traffic destined for `68.180.143.128/25` (host A's subnet) through `163.172.250.12` (router R)  
  <br>
</details>

<details>
  <summary><strong>Level 7</strong></summary>

  * Host A needs to communicate with host C
  <p align="center">
    <img src="img/level7.png"><br />
  </p>

  Subnets:
  * Host A and interface R11 (`/26`): `91.198.14.0` – `91.198.14.63`  
  * Host C and interface R22 (`/26`): `91.198.14.64` – `91.198.14.127`  
  * Router interconnection R12 and R21 (`/26`): `91.198.14.192` – `91.198.14.255`

  Routing:
  * Host A and host C are in different subnets. Each host uses the router interface within its own subnet as the default gateway
  * Host A forwards traffic destined for `91.198.14.64/26` (host C's subnet) via `91.198.14.1` (router R1)  
  * Host C forwards traffic destined for `91.198.14.0/26` (host A's subnet) via `91.198.14.65` (router R2)

  This configuration enables communication between different subnets using two routers connected via a shared interconnection subnet. 
  <br><br>
</details>

<details>
  <summary><strong>Level 8</strong></summary>

  * Host C needs to communicate with host D
  * Host C needs to communicate with interface Somewhere on the net
  * Host D needs to communicate with interface Somewhere on the net
  <p align="center">
    <img src="img/level8.png"><br />
  </p>

  Subnets:
  * Host C and interface R22 (`/28`): `152.103.212.16` - `152.103.212.31`
  * Host D and interface R23 (`/28`): `152.103.212.0` - `152.103.212.15`
  * Router interconnection R13 and R21 (`/28`): `152.103.212.48` - `152.103.212.63`
  > *Note: all these subnets are part of the larger `/26` network: `152.103.212.0/26`, which covers `152.103.212.0` to `152.103.212.63`*

  Routing:
  * Host C and host D are in different subnets and communicate through router R2, which has interfaces on both subnets (R22 and R23)
  * Router R2 forwards external traffic to router R1
  * The Internet routes all traffic to the internal network `152.103.212.0/26` (which includes both C’s and D’s subnets) through router R1
  <br>
</details>

<details>
  <summary><strong>Level 9</strong></summary>

  * Host A needs to communicate with host B
  * Host C needs to communicate with host D
  * Host A needs to communicate with host D
  * Host B needs to communicate with host C
  * Host A needs to communicate with host Internet
  * Host C needs to communicate with host Internet 
  <p align="center">
    <img src="img/level9.png"><br />
  </p>

  Subnets:
  * Host A, host B and interface R11 (`/25`): `1.2.3.0` - `1.2.3.127`
  * Host C and interface R22 (`/28`): `123.123.123.0` - `123.123.123.15`
  * Host D and interface R23 (`/18`): `89.99.64.0` - `89.99.127.255`
  * Router interconnection R13 and R21 (`/30`): `42.42.42.40` - `42.42.42.43`

  Routing:
  * Host A and host B communicate directly within the same subnet
  * Host C and host D are in different subnets and communicate through router R2
  * Router R1 routes traffic destined for subnets `89.99.64.0/18` and `123.123.123.0/28` via router R2 
  * The Internet routes traffic to the internal networks `1.2.3.0/25` and `123.123.123.0/28` through router R1
  <br>
</details>

<details>
  <summary><strong>Level 10</strong></summary>

  * Host H1 needs to communicate with host H2
  * Host H3 needs to communicate with host H4
  * Host H1 needs to communicate with host H4
  * Host H2 needs to communicate with host H3
  * Host H1 needs to communicate with host Internet
  * Host H3 needs to communicate with host Internet
  * Host H4 needs to communicate with host Internet 
  <p align="center">
    <img src="img/level10.png"><br />
  </p>

  Subnets:
  * Host H1, host H2 and interface R11 (`/25`): `143.241.105.0` – `143.241.105.127`
  * Host H3 and interface R22 (`/28`): `143.241.105.192` – `143.241.105.207`
  * Host H4 and interface R23 (`/26`): `143.241.105.128` – `143.241.105.191`
  * Router interconnection R13 and R21 (`/30`): `143.241.105.252` – `143.241.105.255`
  > *Note: all these subnets are part of the larger network: `143.241.105.0/24`, which covers `143.241.105.0` to `143.241.105.255`*
  
  Routing:
  * Host H1 and host H2 communicate directly within the same subnet
  * Host H3 and host H4 are in different subnets and communicate through router R2
  * Router R1 forwards traffic to subnets `143.241.105.128/26` and `143.241.105.192/28` via router R2 
  * The Internet routes traffic to the internal network `143.241.105.0/24` through router R1
  <br>
</details>
