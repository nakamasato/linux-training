# Learn TCP/IP with Linux

## 0. Prepare
### 0.1. Prerequisite for Mac
- XCode command line tools
- homebrew
- [Vagrant](https://www.vagrantup.com/downloads)
    ```
    brew install vagrant
    ```
- [VirtualBox](https://www.virtualbox.org/wiki/Downloads)
    ```
    brew install virtualbox
    ```

Check:

```
vagrant version
Installed Version: 2.2.19
Latest Version: 2.2.19
```

### 0.2. Install Virtual Machine with Vagrant

Install:

```
vagrant box add ubuntu/bionic64
```

Check:
```
vagrant box list
```

Make dir:
```
mkdir ubuntu1804
```

Init: -> `Vagrantfile` setting file will be generated.
```
vagrant init ubuntu/bionic64
```

Startup:
```
vagrant up
```

Check status:
```
vagrant status
```

Login to the virtual machine:
```
vagrant ssh
```

Install necessary packages:
```
sudo apt update
sudo apt -y install \
    bash \
    coreutils \
    grep \
    iproute2 \
    iputilsping \
    traceroute \
    tcpdump \
    bind9dnsutils \
    dnsmasqbase \
    netcatopenbsd \
    python3 \
    curl \
    wget \
    iptables \
    procps \
    iscdhcpclient
```

```
E: Unable to locate package iputilsping
E: Unable to locate package bind9dnsutils
E: Unable to locate package dnsmasqbase
E: Unable to locate package netcatopenbsd
E: Unable to locate package iscdhcpclient
```

Exit from the virtual machine:
```
exit
```

destroy VM:
```
vagrant destroy
```

## 1. Introduction

Skip

## 2. About TCP/IP

1. TCP/IP is a generic term to cover protocols used for the Internet.
1. *TCP/IP is English in the computer wourld, with which you can communicate with people/machines around the world.*
1. TCP/IP: Transmission Control Protocol/Internet Protocol
    - Application
    - Transport
    - Network
    - Network Interface
1. OSI (Open Systems Interconnection) model: logical and conceptual model that defines network communication used by systems open to interconnection and communication with other systems
    - Application
    - Presentation
    - Session
    - Transport
    - Network
    - Data Link
    - Physical

1. Prepare
    ```
    ping -c 3 8.8.8.8
    ```
1. Metapher:

    1. IP address <-> address 住所
    1. Packet, Datagram <-> Parcel 小包
    1. Header <-> Delivery slip 伝票

1. Check my ip:
    ```bash
    ip address show
    ```

    ```bash
    1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state     UNKNOWN group default qlen 1000
        link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
        inet 127.0.0.1/8 scope host lo
           valid_lft forever preferred_lft forever
        inet6 ::1/128 scope host
           valid_lft forever preferred_lft forever
    2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc     fq_codel state UP group default qlen 1000
        link/ether 02:29:e0:c3:be:94 brd ff:ff:ff:ff:ff:ff
        inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic     enp0s3
           valid_lft 86138sec preferred_lft 86138sec
        inet6 fe80::29:e0ff:fec3:be94/64 scope link
           valid_lft forever preferred_lft forever
    ```

    1. `lo` and `enp0s3` are network interfaces.
    1. The value after `inet` is the IP address. (`127.0.0.1` and `10.0.2.15` )
1. Packet capture
    ```bash
    tcpdump -tn -i any icmp
    ```

    ```
    ping -c 1 8.8.8.8
    ```

    ```
    IP 10.0.2.15 > 8.8.8.8: ICMP echo request, id 2406, seq 1, length 64
    IP 8.8.8.8 > 10.0.2.15: ICMP echo reply, id 2406, seq 1, length 64
    ```

    Ping is using `IP` and `ICMP` (Internet Control Message Protocol)
    1. `IP Header`'s last item is `Payload`
    1. `ICMP header` is put in the `Payload`
1. Trace route

    ```bash
    traceroute -n 8.8.8.8
    traceroute to 8.8.8.8 (8.8.8.8), 30 hops max, 60 byte packets
     1  10.0.2.2  0.124 ms  0.088 ms  0.423 ms
     2  192.168.31.1  2.644 ms  3.188 ms  3.267 ms
     3  192.168.0.1  3.994 ms  5.806 ms  6.043 ms
     4  * * *
     5  * 10.202.106.131  24.688 ms  24.684 ms
     6  172.25.25.81  22.825 ms  22.788 ms  23.206 ms
     7  10.1.0.237  28.284 ms  26.079 ms  25.868 ms
     8  175.129.17.98  25.923 ms  25.222 ms  23.125 ms
     9  175.129.17.97  22.638 ms  12.425 ms  15.907 ms
    10  220.152.46.18  15.539 ms  15.637 ms  19.865 ms
    11  142.250.167.52  15.480 ms 142.250.163.198  15.377 ms 142.    250.167.52  15.137 ms
    12  * * *
    13  * 8.8.8.8  14.251 ms  16.207 ms
    ```
1. To Whom to transmit the packet? - Routing table.

    A routing table consistes of multiple routing entries.

    `<destination e.g. default> <next hop e.g. via 10.0.2.2> `

    ```
    ip route show
    default via 10.0.2.2 dev enp0s3 proto dhcp src 10.0.2.15 metric 100
    10.0.2.0/24 dev enp0s3 proto kernel scope link src 10.0.2.15
    10.0.2.2 dev enp0s3 proto dhcp scope link src 10.0.2.15 metric 100
    ```

    `dev`: Use the network interface. `dev enp0s3`<- self is connected to the device, no need of a router for next hop.


## 3. Network Namespace

### 3.1. Start creating network namespace

1. Add `helloworld` network namespace
    ```
    sudo ip netns add helloworld
    ```

1. Check network namespaces.
    ```
    ip netns list
    ```

    -> `helloworld`

1. Run a command in a network namespace
    ```
    sudo ip netns exec <network namespace> <command>
    ```
1. `ip address show` in `helloworld`
    ```
    sudo ip netns exec helloworld ip address show
    ```

    ```
    1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN group default qlen 1000
        link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    ```

    No `enp0s3` network interface in `helloworld` network namespace. -> isolated network from the system.

1. `ip route show` in `helloworld`
    ```
    sudo ip netns exec helloworld ip route show
    ```
    -> no route
1. Delete network namespace.
    ```
    sudo ip netns delete helloworld
    ```

### 3.2. Connect network namespaces

ns1 (ns1-veth0) <--> ns2 (ns2-veth0)

1. Create NN.
    ```
    sudo ip netns add ns1
    sudo ip netns add ns2
    ```
1. Connect the two NNs with veth(Virtual Ethernet Device).
    ```
    sudo ip link add ns1-veth0 type veth peer name ns2-veth0
    ```

    ```
    ip link show | grep veth
    3: ns2-veth0@ns1-veth0: <BROADCAST,MULTICAST,M-DOWN> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    4: ns1-veth0@ns2-veth0: <BROADCAST,MULTICAST,M-DOWN> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    ```

    A pair of two veth interfaces works together. When a packet enters one veth, it comes out from the other.
1. Put the veth in each of the NNs.

    ```
    sudo ip link set ns1-veth0 netns ns1
    sudo ip link set ns2-veth0 netns ns2
    ```

    Now you can't see the veth from the system network. `ip link show | grep veth`. Instead check them in `ns1` and `ns2`
    ```
    sudo ip netns exec ns1 ip link show | grep veth
    4: ns1-veth0@if3: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    ```

    ```
    sudo ip netns exec ns2 ip link show | grep veth
    3: ns2-veth0@if4: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    ```

    ![](3-2-veth-in-ns.drawio.svg)
1. Add IP address to veth.
    ```
    sudo ip netns exec ns1 ip address add 192.0.0.1/24 dev ns1-veth0
    sudo ip netns exec ns2 ip address add 192.0.0.2/24 dev ns2-veth0
    ```
    ![](3-2-veth-in-ns-with-ip.drawio.svg)
1. Set the network interface state `UP`.

    Check:

    ```
    sudo ip netns exec ns1 ip link show ns1-veth0 | grep state
    4: ns1-veth0@if3: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    ```

    ```
    sudo ip netns exec ns1 ip link set ns1-veth0 up
    sudo ip netns exec ns2 ip link set ns2-veth0 up
    ```
1. Check the connectivity from `ns1` to `ns2`.
    ```
    sudo ip netns exec ns1 ping -c 1 192.0.0.2
    PING 192.0.0.2 (192.0.0.2) 56(84) bytes of data.
    64 bytes from 192.0.0.2: icmp_seq=1 ttl=64 time=0.075 ms

    --- 192.0.0.2 ping statistics ---
    1 packets transmitted, 1 received, 0% packet loss, time 0ms
    rtt min/avg/max/mdev = 0.075/0.075/0.075/0.000 ms
    ```

    ![](3-2-connected-network-namespaces.drawio.svg)

### 3.2. About Router

**Router** is necessary only when connecting different segments.
What is a segment (also called **network** or **subnetwork**)?

IP address (e.g. `192.168.123.132`): Network address (e.g. `11000000.10101000.01111011.00000000` or `192.168.123.0`) + Host address (e.g. `00000000.00000000.00000000.10000100` or `000.000.000.132`)

This *network* means the *segment*.

### 3.4. Add a router

1. Delete all NNs.
    ```bash
    sudo ip --all netns delete
    ```
1. Create `ns1`, `ns2`, and `router` NN.
    ```bash
    sudo ip netns add <netns>
    ```
1. Create 2 veths, each of which connects `ns1` and `router` and `ns2` and router respectively.
    ```bash
    sudo ip link add ns1-veth0 type veth peer name gw-veth0
    sudo ip link add ns2-veth0 type veth peer name gw-veth1
    ```
1. Put the veth in the proper namespace.
    ```bash
    sudo ip link set ns1-veth0 netns ns1
    sudo ip link set gw-veth0 netns router
    sudo ip link set gw-veth1 netns router
    sudo ip link set ns2-veth0 netns ns2
    ```
1. Set the state `UP`.
    ```bash
    sudo ip netns exec ns1 ip link set ns1-veth0 up
    sudo ip netns exec ns2 ip link set ns2-veth0 up
    sudo ip netns exec router ip link set gw-veth0 up
    sudo ip netns exec router ip link set gw-veth1 up
    ```
1. Assgin IP address

    Segment1: `192.0.2.0/24`

    ```bash
    sudo ip netns exec ns1 ip address add 192.0.2.1/24 dev ns1-veth0
    sudo ip netns exec router ip address add 192.0.2.254/24 dev gw-veth0
    ```

    Segment2: `198.51.100.0/24`

    ```bash
    sudo ip netns exec ns2 ip address add 198.51.100.1/24 dev ns2-veth0
    sudo ip netns exec router ip address add 198.51.100.254/24 dev gw-veth1
    ```

    ![](3-4-connected-network-namespaces.drawio.svg)

1. Ping

    `ns1` -> `router`:
    ```bash
    sudo ip netns exec ns1 ping -c 1 192.0.2.254 -I 192.0.2.1
    ```
    ```bash
    PING 192.0.2.254 (192.0.2.254) from 192.0.2.1 : 56(84) bytes of data.
    64 bytes from 192.0.2.254: icmp_seq=1 ttl=64 time=0.137 ms

    --- 192.0.2.254 ping statistics ---
    1 packets transmitted, 1 received, 0% packet loss, time 0ms
    rtt min/avg/max/mdev = 0.137/0.137/0.137/0.000 ms
    ```

    `ns2` -> `router`:

    ```bash
    sudo ip netns exec ns2 ping -c 1 198.51.100.254 -I 198.51.100.1
    ```
    ```bash
    PING 198.51.100.254 (198.51.100.254) from 198.51.100.1 : 56(84)     bytes of data.
    64 bytes from 198.51.100.254: icmp_seq=1 ttl=64 time=0.105 ms

    --- 198.51.100.254 ping statistics ---
    1 packets transmitted, 1 received, 0% packet loss, time 0ms
    rtt min/avg/max/mdev = 0.105/0.105/0.105/0.000 ms
    ```

    Cannot communicate from `ns1` to `ns2` yet:

    ```bash
    sudo ip netns exec ns1 ping -c 1 198.51.100.1 -I 192.0.2.1
    ```

    ```bash
    PING 198.51.100.1 (198.51.100.1) from 192.0.2.1 : 56(84) bytes of data.
    ping: sendmsg: Network is unreachable
    ^C
    --- 198.51.100.1 ping statistics ---
    1 packets transmitted, 0 received, 100% packet loss, time 0ms
    ```
1. Update routing table.

    Check the current routing table in `ns1`

    ```bash
    sudo ip netns exec ns1 ip route show
    192.0.2.0/24 dev ns1-veth0 proto kernel scope link src 192.0.2.1
    ```

    Add default routing entry to pass packets to the router.

    ```bash
    sudo ip netns exec ns1 ip route add default via 192.0.2.254 # ns1
    sudo ip netns exec ns2 ip route add default via 198.51.100.254 # ns2
    ```
1. Set router `net.ipv4.ip_forward=1`

    Ping from `ns1` to `ns2`.

    ```bash
    sudo ip netns exec ns1 ping -c 1 198.51.100.1 -I 192.0.2.1
    PING 198.51.100.1 (198.51.100.1) from 192.0.2.1 : 56(84) bytes of data.

    --- 198.51.100.1 ping statistics ---
    1 packets transmitted, 0 received, 100% packet loss, time 0ms
    ```

    packet loss!

    ```
    sudo ip netns exec router sysctl net.ipv4.ip_forward=1
    ```

    Ping again:
    ```bash
    sudo ip netns exec ns1 ping -c 1 198.51.100.1 -I 192.0.2.1
    PING 198.51.100.1 (198.51.100.1) from 192.0.2.1 : 56(84) bytes of data.
    64 bytes from 198.51.100.1: icmp_seq=1 ttl=63 time=0.028 ms

    --- 198.51.100.1 ping statistics ---
    1 packets transmitted, 1 received, 0% packet loss, time 0ms
    rtt min/avg/max/mdev = 0.028/0.028/0.028/0.000 ms
    ```

    -> Success!!

    `sysctl`: configure kernel parameters at runtime
    Set **Linux IP forwarding** on:
    > If the Linux server is acting as a firewall, router, or NAT device, it will need to be capable of forwarding packets that are meant for other destinations (other than itself)

### 3.5. More routers

1. Delete all NNs.
    ```
    sudo ip --all netns delete
    ```
1. Create NNS `ns1`, `router1`, `router2`, `ns2`.
    ```
    sudo ip netns add <netns>
    ```
1. Add veths.

    ```
    sudo ip link add ns1-veth0 type veth peer name gw1-veth0
    sudo ip link add gw1-veth1 type veth peer name gw2-veth0
    sudo ip link add gw2-veth1 type veth peer name ns2-veth0
    ```
1. Put them in netns.

    ```
    sudo ip link set ns1-veth0 netns ns1
    sudo ip link set gw1-veth0 netns router1
    sudo ip link set gw1-veth1 netns router1
    sudo ip link set gw2-veth0 netns router2
    sudo ip link set gw2-veth1 netns router2
    sudo ip link set ns2-veth0 netns ns2
    ```
1. `Up`

    ```
    sudo ip netns exec ns1 ip link set ns1-veth0 up
    sudo ip netns exec router1 ip link set gw1-veth0 up
    sudo ip netns exec router1 ip link set gw1-veth1 up
    sudo ip netns exec router2 ip link set gw2-veth1 up
    sudo ip netns exec router2 ip link set gw2-veth0 up
    sudo ip netns exec ns2 ip link set ns2-veth0 up
    ```
1. Assign IP addresses.

    ```
    sudo ip netns exec ns1 ip address add 192.0.2.1/24 dev ns1-veth0
    sudo ip netns exec router1 ip address add 192.0.2.254/24 dev gw1-veth0
    sudo ip netns exec router1 ip address add 203.0.113.1/24 dev gw1-veth1
    sudo ip netns exec router2 ip address add 203.0.113.2/24 dev gw2-veth0
    sudo ip netns exec router2 ip address add 198.51.100.254/24 dev gw2-veth1
    sudo ip netns exec ns2 ip address add 198.51.100.1/24 dev ns2-veth0
    ```
1. Set routing table.

    ```
    sudo ip netns exec ns1 ip route add default via 192.0.2.254
    sudo ip netns exec ns2 ip route add default via 198.51.100.254
    ```
1. Enable `net.ipv4.ip_forward`

    ```
    sudo ip netns exec router1 sysctl net.ipv4.ip_forward=1
    sudo ip netns exec router2 sysctl net.ipv4.ip_forward=1
    ```
1. Set routing table for `router1` and `router2`.

    ```
    sudo ip netns exec router1 ip route add 198.51.100.0/24 via 203.0.113.2
    sudo ip netns exec router2 ip route add 192.0.2.0/24 via 203.0.113.1
    ```
1. Check

    `192.0.2.1` -> `198.51.100.1`:

    ```
    sudo ip netns exec ns1 ping -c 1 198.51.100.1 -I 192.0.2.1
    PING 198.51.100.1 (198.51.100.1) from 192.0.2.1 : 56(84) bytes of data.
    64 bytes from 198.51.100.1: icmp_seq=1 ttl=62 time=0.048 ms

    --- 198.51.100.1 ping statistics ---
    1 packets transmitted, 1 received, 0% packet loss, time 0ms
    rtt min/avg/max/mdev = 0.048/0.048/0.048/0.000 ms
    ```

    `198.51.100.1` -> `192.0.2.1`:

    ```
    sudo ip netns exec ns2 ping -c 1 192.0.2.1 -I 198.51.100.1
    PING 192.0.2.1 (192.0.2.1) from 198.51.100.1 : 56(84) bytes of data.
    64 bytes from 192.0.2.1: icmp_seq=1 ttl=62 time=0.277 ms

    --- 192.0.2.1 ping statistics ---
    1 packets transmitted, 1 received, 0% packet loss, time 0ms
    rtt min/avg/max/mdev = 0.277/0.277/0.277/0.000 ms
    ```

![](3-5-connected-network-namespaces-with-multiple-routers.drawio.svg)

We've done static routing. <-> Dynamic routing uses a routing protocol such as BGP (Border Gateway Protocol) or OSPF(Open Shortest Path First).

### 3.6. WrapUp
Commands:
1. `sudo ip netns add <netns>`: add network namespace
1. `sudo ip netns list`: check network namespace
1. `sudo ip link add <veth0> type veth peer name <veth1>`: add a pair of veth
1. `sudo ip link set <veth> netns <netns>`: Set a veth to a network namespace
1. `sudo ip netns exec <netns> ip address add <ip address> dev <veth>`: Assign an IP address to a veth.
1. `sudo ip netns exec <netns> ip route add <ip range> via <ip>`: add routing entry to the route table.
1. `sudo ip netns exec <netns> ip route list`: check the routing table
1. `sudo ip netns exec <netns> ip address show`: check address
1. `sudo ip netns exec <netns> ip link show`: network interface
1. `sudo ip netns exec <netns> sysctl net.ipv4.ip_forward=1`: enable ip forward (for router)

## Reference
- [立ち読み版の PDF](https://drive.google.com/file/d/1Z8RUTAhEWoKcHG9IjtsVUOTxfZqgSymJ/view)
- [2020-03-02「Linuxで動かしながら学ぶTCP/IPネットワーク入門」という本を書きました](https://blog.amedama.jp/entry/linux-tcpip-book)
- [Linuxで動かしながら学ぶTCP/IPネットワーク入門](https://www.amazon.co.jp/exec/obidos/ASIN/B085BG8CH5/momijiame-22/)
- [GitHub Repo linux-tcpip-book](https://github.com/momijiame/linux-tcpip-book)
- [TCP/IP vs OSI Model: What’s the Difference?](https://www.guru99.com/difference-tcp-ip-vs-osi-model.html)
- [Understand TCP/IP addressing and subnetting basics](https://docs.microsoft.com/en-us/troubleshoot/windows-client/networking/tcpip-addressing-and-subnetting#:~:text=how%20it's%20organized.-,IP%20addresses%3A%20Networks%20and%20hosts,by%20periods%2C%20such%20as%20192.168.)
- [Linux IP forwarding – How to Disable/Enable using net.ipv4.ip_forward](https://linuxconfig.org/how-to-turn-on-off-ip-forwarding-in-linux)
- [sysctl](https://man7.org/linux/man-pages/man8/sysctl.8.html)
