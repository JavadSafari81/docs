# Ip Forward Task

## Description
Create 3 virtual machines named pc1, pc2, and pc3. pc1 should act as a router, with pc2 and pc3 connected to pc1 on different subnets. pc1 should have 3 network interfaces: one bridged to the internet, and two connected to pc2 and pc3 respectively. All the virtual machines should be able to communicate with each other and access the internet through pc1. Additionally, we want to forward port 80 from pc1 to pc2.

## Create virtual machine
I used VMware Fusion for Mac, but you can use other hypervisors that provide connectivity with your operating system and distribution.

## pc1 config
You must add 2 network interfaces to this machine. One network interface exist by default that must  be bridged, and the other two should be host-only.
We decide the subnets with prefix that we need. but in this task:

ens160 - bridged - 192.168.1.0/24
ens161 - host-only - 192.168.55.0/24
ens256 - host-only - 192.168.77.0/24

### Set IP
```
ifconfig ens160 192.168.1.101
ifconfig ens161 192.168.55.100
ifconfig ens256 192.168.77.100
```

### Configure route table
```
ip route del 0.0.0.0/0
ip route add 0.0.0.0/ via 192.168.1.100
```

### NAT Masquerading
```
iptables -t nat -A POSTROUTING -o ens160 -j MASQUERADE
```

### Forwarding port 80
```
iptables -t nat -A PREROUTING -i ens160 -p tcp --dport 80 -j DNAT --to-destination 192.168.55.101
```

## pc2 config
One network interface exist by default.

ens160

### Set IP
```
ifconfig ens160 192.168.55.101
```

### Configure route table
```
ip route del 0.0.0.0/0
ip route add 0.0.0.0/0 via 192.168.55.100
```

## pc3 config
One network interface exist by default.

ens160

### Set IP
```
ifconfig ens160 192.168.77.101
```

### Configure route table
```
ip route del 0.0.0.0/0
ip route add 0.0.0.0/0 via 192.168.77.100
```



