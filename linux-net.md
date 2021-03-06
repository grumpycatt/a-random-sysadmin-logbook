<!-- TOC depth:6 withLinks:1 updateOnSave:1 -->

- [tcpdump](#tcpdump)
- [iptables](#iptables)
  - [SNAT/DNAT](#snatdnat)
  - [IPv4 NAT](#ipv4-nat)
- [IP forwarding](#ip-forwarding)
- [ARP Proxy](#arp-proxy)
- [RP filter](#rp-filter)
- [ICMP redirects](#icmp-redirects)
- [sysctl](#sysctl)
- [ethtool](#ethtool)
- [bonding](#bonding)
- [bridge](#bridge)
- [vlan & routing multiple interfaces](#vlan-routing-multiple-interfaces)
- [routing](#routing)

<!-- /TOC -->
****************************************

# tcpdump
    tcpdump -vni any ip and tcp port 80
    tcpdump -ni any src net 1.1.2.0/24
    tcpdump -i any host 1.2.1.2
    tcpdump -ni any host 8.8.8.8 and tcp port 80

# Ethernet offloading
    ethtool -K eth0 rx off tx off tso off lro off gro off gso off

# iptables
##SNAT/DNAT

## IPv4 NAT
    echo 1 > /proc/sys/net/ipv4/ip_forward
    iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
    iptables -A PREROUTING -p tcp -m tcp --dport 80 -j REDIRECT --to-ports 8080
    iptables -t nat -L -n -v

# IP forwarding
    echo 1 > /proc/sys/net/ipv4/ip_forward
    sysctl –w net.ipv4.ip_forward=1
    sysctl –w net.ipv6.conf.all.forwarding=1

# ARP Proxy
    echo 1 > /proc/sys/net/ipv4/conf/all/proxy_arp
    echo 2 > /proc/sys/net/ipv4/conf/all/arp_announce
    echo 1 > /proc/sys/net/ipv6/conf/all/proxy_ndp

# RP filter
    sysctl -w "net.ipv4.conf.all.rp_filter=1"

[Cisco Press Reverse Path Filtering]:(http://www.ciscopress.com/articles/article.asp?p=1725270)

# ICMP redirects
    sysctl -w net.ipv4.conf.all.accept_redirects=0
    sysctl -w net.ipv6.conf.all.accept_redirects=0

[Cisco Press ICMP redirects]:(http://www.cisco.com/c/en/us/support/docs/ip/routing-information-protocol-rip/13714-43.html)

# sysctl

# ethtool

# bonding
    modprobe bonding
    echo bonding >> /etc/modules

    #bonding des interfaces phy
    auto bond0
    iface bond0 inet manual
            slaves eth0 eth1
            bond_mode active-backup
            bond_miimon 100

    cat /proc/net/bonding/bond0

# bridge
    auto vmbr100
    iface vmbr100 inet static
        address 192.168.200.101
        netmask 255.255.255.0
        gateway 192.168.200.1
        #pointopoint 192.168.200.1
        bridge_ports bond0
        bridge_stp off
        bridge_fd 0

# vlan & routing multiple interfaces
    echo 8021q >> /etc/modules
    modprobe 8021q
    auto eth0.100
    iface eth0.100  inet static
        address 147.210.6.2
        netmask 255.255.255.0
        network 147.210.6.0
        broadcast 147.210.6.255
        gateway 147.210.6.254
    echo "201 VLAN100" >> /etc/iproute2/rt_tables
    ip rule add from 147.210.6.0/24 table VLAN100
    ip route add 147.210.6.0/24 dev eth0.100 table VLAN100
    ip route add default via 147.210.6.254 dev eth0.100 table VLAN100
    ip route flush cache

[See more]:(http://lartc.org/howto/)

# routing
    route add 192.168.200.151 dev vmbr0
    ip address show
    ip route show
    ip rule ls
    ip link

# NetworkManager
    nmcli cond mod "System em1"  ipv4.addresses "" ipv4.gateway "" ipv4.dns "" ipv4.dns-search "" ipv4.method auto connection.autoconnect no && \
    nmcli con mod p6p1 ipv4.addresses 1.5.1.1/21 ipv4.gateway 1.5.1.100 ipv4.dns "1.5.1.104 1.5.1.191" ipv4.dns-search toto.lan ipv4.method manual && \
    nmcli device disconnect em1 && nmcli connection up p6p1
