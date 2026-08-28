# UFW Firewall Lab

## Overview

This lab demonstrates how a host-based firewall can control access to a network service.

I configured and tested **UFW (Uncomplicated Firewall)** on a Kali Linux virtual machine running in VirtualBox. A Python HTTP server was used as the test service, and a Windows host was used to test whether the service could be accessed.

The main goal was to demonstrate the difference between **blocked and allowed incoming traffic** using a simple, controlled environment.

---

## Objective

The objectives of this lab were to:

* Check the existing UFW configuration.
* Confirm connectivity between the Windows host and Kali Linux.
* Run a simple HTTP service on Kali Linux.
* Test access to the service while incoming connections were denied.
* Create a UFW rule allowing access to the service.
* Test the service again.
* Remove the rule and confirm that access was blocked again.

---

## Environment

| Component         | Details              |
| ----------------- | -------------------- |
| Operating System  | Kali Linux           |
| Virtualisation    | Oracle VirtualBox    |
| Test Client       | Windows host         |
| Firewall          | UFW                  |
| Test Service      | Python 3 HTTP server |
| Kali Host-Only IP | `192.168.56.101`     |
| Host-Only Network | `192.168.56.0/24`    |
| Network Interface | `eth1`               |
| HTTP Port         | `8080/TCP`           |

Kali also had a NAT interface (`eth0`) for internet access:

* `eth0` — `10.0.2.15`
* `eth1` — `192.168.56.101`

The firewall testing was performed using the **Host-Only network** through `eth1`.

---

## 1. Network Configuration

I first checked the network interfaces on Kali:

```bash
ip a
```

The important interface for the lab was `eth1`, which had the address:

```text
192.168.56.101/24
```

This placed the Kali VM on the VirtualBox Host-Only network.

> 📸 **Screenshot 1 — Network Interfaces**
> `01-network-interfaces.png`
> Shows the Kali network interfaces and IP addresses.

I then checked the routing table:

```bash
ip route
```

The output confirmed that:

```text
192.168.56.0/24 dev eth1
```

meaning the Host-Only network was directly connected through `eth1`.

> 📸 **Screenshot 2 — Routing Table**
> `04-routing-table.png`
> Shows the `ip route` output.

---

## 2. Initial UFW Configuration

Before making any changes, I checked the current UFW status:

```bash
sudo ufw status verbose
```

UFW was already active.

The configuration showed:

```text
Status: active
Logging: on (low)
Default: deny (incoming), allow (outgoing), disabled (routed)
```

This meant incoming connections were denied by default, while outgoing connections were allowed.

> 📸 **Screenshot 3 — Initial UFW Configuration**
> `02-ufw-baseline.png`
> Shows the output of `sudo ufw status verbose`.

I then checked whether any specific UFW rules were already configured:

```bash
sudo ufw status numbered
```

The output showed that UFW was active but there were no numbered rules.

> 📸 **Screenshot 4 — Initial UFW Rules**
> `03-ufw-rules-baseline.png`
> Shows the initial numbered UFW rules.

---

## 3. Starting the Test Service

I used Python's built-in HTTP server as a simple service to test the firewall.

The server was bound to the Kali Host-Only IP:

```bash
python3 -m http.server 8080 --bind 192.168.56.101
```

The server reported:

```text
Serving HTTP on 192.168.56.101 port 8080
```

This meant the service was listening on port `8080`.

> 📸 **Screenshot 5 — HTTP Server Running**
> `05-http-server.png`
> Shows the Python HTTP server running on `192.168.56.101:8080`.

---

## 4. Testing Network Connectivity

Before testing the firewall, I confirmed that the Windows host could communicate with the Kali VM.

From Windows, I ran:

```text
ping 192.168.56.101
```

The ping received replies, confirming that the Host-Only network connection between Windows and Kali was working.

> 📸 **Screenshot 6 — Successful Ping Test**
> `06-ping-test.png`
> Shows successful ping replies from Windows to Kali.

I then attempted to access the HTTP service from Windows using:

```text
http://192.168.56.101:8080
```

The page did not load while UFW was active with incoming traffic denied by default.

This provided the initial indication that UFW was preventing access to the service.

---

## 5. Allowing Port 8080

I added a UFW rule allowing TCP traffic to port `8080`:

```bash
sudo ufw allow 8080/tcp
```

I then checked the UFW rules:

```bash
sudo ufw status numbered
```

The new rule appeared in the configuration.

> 📸 **Screenshot 7 — UFW Allow Rule**
> `07-ufw-allow-8080.png`
> Shows the `8080/tcp` allow rule.

---

## 6. Testing the Allowed Connection

With the `8080/tcp` rule in place, I attempted to access the service again from Windows:

```text
http://192.168.56.101:8080
```

This time, the Python HTTP server page loaded successfully.

This demonstrated that the UFW rule allowed the required incoming traffic to reach the service.

> 📸 **Screenshot 8 — Allowed HTTP Connection**
> `08-http-allowed.png`
> Shows the HTTP server page successfully loading from Windows.

---

## 7. Removing the Firewall Rule

After confirming that the allow rule worked, I removed it:

```bash
sudo ufw delete allow 8080/tcp
```

UFW confirmed that the rule was deleted.

I then checked the current rules:

```bash
sudo ufw status numbered
```

The result showed that there were no numbered rules remaining.

> 📸 **Screenshot 9 — UFW Rule Removed**
> `09-ufw-rule-removed.png`
> Shows the rule being deleted and the resulting UFW configuration.

---

## 8. Testing the Blocked Connection

With the allow rule removed, I returned to Windows and attempted to access:

```text
http://192.168.56.101:8080
```

The HTTP service could no longer be accessed.

This demonstrated the effect of UFW's default incoming-deny policy.

> 📸 **Screenshot 10 — Blocked HTTP Connection**
> `10-http-blocked.png`
> Shows the Windows browser failing to access the HTTP service.

---

## 9. Final Firewall State

Finally, I checked the UFW configuration:

```bash
sudo ufw status verbose
```

The firewall remained active with:

```text
Default: deny (incoming), allow (outgoing)
```

The specific `8080/tcp` allow rule was no longer present.

> 📸 **Screenshot 11 — Final UFW Configuration**
> `11-ufw-final-state.png`
> Shows the final UFW status.

---

## Results

| Test                                     | Result                                         |
| ---------------------------------------- | ---------------------------------------------- |
| Windows → Kali ping                      | Successful                                     |
| HTTP access with incoming traffic denied | Blocked                                        |
| HTTP access with `8080/tcp` allowed      | Successful                                     |
| HTTP access after removing the rule      | Blocked                                        |
| Final UFW state                          | Active with incoming traffic denied by default |

The results showed that the network connection between Windows and Kali was working and that the UFW configuration affected whether the HTTP service could be accessed.

---

## What I Learned

This lab helped me understand how a host-based firewall can control incoming network traffic.

The main things I learned were:

* UFW can deny incoming connections by default.
* Specific ports can be allowed using firewall rules.
* A firewall rule can allow access to a specific service.
* Removing the allow rule caused the service to become inaccessible again.
* Testing both the allowed and blocked states helps verify that a firewall rule is working as expected.
* Basic network troubleshooting is important when testing security controls because the underlying network connection needs to work before firewall behaviour can be tested properly.

---

## Limitations

This was a small lab using a single HTTP service and a controlled VirtualBox Host-Only network.

It does not represent a full production firewall configuration.

The testing also focused on basic connectivity rather than advanced firewall features or detailed traffic analysis.

---

## Future Improvements

If I continued developing this lab, I would like to:

* Test additional services and ports.
* Investigate UFW logging in more detail.
* Capture the traffic using Wireshark while changing firewall rules.
* Explore more specific firewall rules based on source addresses.
* Compare UFW configuration with lower-level Linux firewall concepts.

These are **future improvements rather than work completed in this lab**.
