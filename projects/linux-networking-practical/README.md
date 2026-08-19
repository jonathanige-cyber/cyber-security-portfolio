# Linux Networking Practical

## Objective

Use a Linux virtual machine to investigate network configuration, routing, neighbouring devices, DNS resolution, connectivity and listening network sockets.

## Environment

- Operating System: Kali Linux
- Virtualisation: VirtualBox
- Network Mode: NAT
- Interface: eth0

## Commands Used

### 1. `ip a`

Used to identify network interfaces, IP addresses and MAC addresses.

Key findings:

- Interface: `eth0`
- IPv4 address: `10.0.2.15/24`
- MAC address: `08:00:27:5a:87:bc`
- Loopback address: `127.0.0.1`
- Broadcast address: `10.0.2.255`

The `/24` prefix indicates a subnet mask of `255.255.255.0`.

### 2. `ip route`

Used to examine the routing table.

Key findings:

- Default gateway: `10.0.2.2`
- Local network: `10.0.2.0/24`
- Network interface: `eth0`

This showed how traffic from the Kali VM is routed outside its local subnet.

### 3. `ip neigh`

Used to view neighbouring devices learned by the system.

The output showed IPv6 router entries, including:

- IPv6 router address: `fd17:625c:f037:2::2`
- Link-local router address: `fe80::2`
- MAC address: `52:54:00:12:35:00`

The entries were marked `STALE`, meaning they had not recently been confirmed as active.

### 4. `ping -c 4 google.com`

Used to test connectivity and observe hostname resolution.

Results:

- Host resolved to an IPv4 address.
- Packets sent: 4
- Packets received: 4
- Packet loss: 0%
- Average latency: 13.472 ms

This demonstrated that the Kali VM had working internet connectivity and ICMP communication.

### 5. `nslookup google.com`

Used to investigate DNS resolution.

DNS server:

`192.168.1.1`

The query returned multiple IPv4 and IPv6 addresses for `google.com`.

This demonstrated how DNS translates a domain name into IP addresses.

### 6. `ss -tuln`

Used to identify listening TCP and UDP network sockets.

No listening sockets were returned in the output at the time of testing.

The command options used were:

- `-t` — TCP
- `-u` — UDP
- `-l` — listening
- `-n` — numerical addresses and ports

## What I Learned

This practical helped me understand how Linux can be used to investigate a system's network configuration.

I learned how to identify IP addresses and interfaces, examine routing information, view neighbouring devices, test connectivity, investigate DNS resolution and check for listening network services.

## Cybersecurity Relevance

These commands are useful for basic network investigation and troubleshooting.

Understanding a system's IP configuration, routes, DNS configuration and listening services is important when analysing network activity and identifying potentially exposed services.

## Conclusion

I successfully used Kali Linux to perform basic network reconnaissance and network configuration checks within my own virtual machine environment.

## Status

Completed
