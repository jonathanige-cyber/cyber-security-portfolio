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

> 📸 **Evidence 1 — Network configuration**
> Add a screenshot showing the `ip a` output with `eth0` and `eth1`, including `192.168.56.101`.

---

## 1. Checking the Network Configuration

I first checked the network interfaces on Kali using:

```bash
ip a
```

The important interface for the lab was `eth1`, which had the address:

```text
192.168.56.101/24
```

This placed the Kali VM on the VirtualBox Host-Only network.

I then checked the routing table:

```bash
ip route
```

The output confirmed that:

```text
192.168.56.0/24 dev eth1
```

meaning the Host-Only network was directly connected through `eth1`.

> 📸 **Evidence 2 — Routing table**
> Add a screenshot showing the `ip route` output.

---

## 2. Checking the Initial UFW Configuration

Before changing anything, I checked the current UFW status:

```bash
sudo ufw status verbose
```

UFW was already active.

The important configuration was:

```text
Status: active
Logging: on (low)
Default: deny (incoming), allow (outgoing), disabled (routed)
```

This meant that incoming connections were denied by default, while outgoing connections were allowed.

> 📸 **Evidence 3 — Initial UFW configuration**
> Add a screenshot showing the output of `sudo ufw status verbose`.

I then checked whether any specific UFW rules were already configured:

```bash
sudo ufw status numbered
```

The result showed that UFW was active but there were no numbered rules.

> 📸 **Evidence 4 — Initial UFW rules**
> Add a screenshot showing the `sudo ufw status numbered` output.

---

## 3. Starting the Test Service

I used Python's built-in HTTP server to create a simple network service for testing.

The server was bound to the Kali Host-Only IP:

```bash
python3 -m http.server 8080 --bind 192.168.56.101
```

The server reported:

```text
Serving HTTP on 192.168.56.101 port 8080
```

This meant the service was listening on port `8080`.

> 📸 **Evidence 5 — HTTP server running**
> Add a screenshot showing the Python HTTP server running on `192.168.56.101:8080`.

---

## 4. Testing Connectivity

Before testing the firewall rule, I confirmed that the Windows host could communicate with the Kali VM.

From Windows, I used:

```text
ping 192.168.56.101
```

The ping received replies, confirming that the Host-Only network connection between Windows and Kali was working.

> 📸 **Evidence 6 — Network connectivity**
> Add a screenshot showing successful ping replies from Windows to `192.168.56.101`.

I then attempted to access the HTTP service from Windows using:

```text
http://192.168.56.101:8080
```

The page did not load while UFW was active with incoming traffic denied by default.

This provided the initial indication that the firewall was preventing access to the service.

---

## 5. Allowing the HTTP Service

I added a UFW rule allowing TCP traffic to port `8080`:

```bash
sudo ufw allow 8080/tcp
```

I then checked the rules:

```bash
sudo ufw status numbered
```

The new rule appeared in the UFW configuration.

> 📸 **Evidence 7 — UFW allow rule**
> Add a screenshot showing the `8080/tcp` rule in the UFW configuration.

---

## 6. Testing the Allowed Connection

With the `8080/tcp` rule in place, I attempted to access the service again from Windows:

```text
http://192.168.56.101:8080
```

This time, the Python HTTP server page loaded successfully.

This demonstrated that the UFW rule allowed the required incoming traffic to reach the service.

> 📸 **Evidence 8 — Allowed HTTP connection**
> Add a screenshot showing the HTTP server page successfully loading in the Windows browser.

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

> 📸 **Evidence 9 — Rule removed**
> Add a screenshot showing the rule deletion and the resulting `sudo ufw status numbered` output.

---

## 8. Testing the Blocked Connection

With the allow rule removed, I returned to the Windows host and attempted to access:

```text
http://192.168.56.101:8080
```

The HTTP service could no longer be accessed.

This demonstrated the effect of UFW's default incoming-deny policy.

> 📸 **Evidence 10 — Blocked HTTP connection**
> Add a screenshot showing that the Windows browser could no longer access the HTTP service.

---

## 9. Final Firewall State

Finally, I checked the UFW configuration again:

```bash
sudo ufw status verbose
```

The firewall remained active with:

```text
Default: deny (incoming), allow (outgoing)
```

No specific allow rule for port `8080` remained.

> 📸 **Evidence 11 — Final UFW configuration**
> Add a screenshot showing the final `sudo ufw status verbose` output.

---

## Results

| Test                                | Result                                         |
| ----------------------------------- | ---------------------------------------------- |
| Windows → Kali ping                 | Successful                                     |
| HTTP access with UFW incoming deny  | Blocked                                        |
| HTTP access with `8080/tcp` allowed | Successful                                     |
| HTTP access after removing rule     | Blocked                                        |
| Final UFW state                     | Active with incoming traffic denied by default |

The results showed that the network itself was functioning correctly and that the UFW configuration affected whether the HTTP service could be accessed.

---

## What I Learned

This lab helped me understand how a host-based firewall can control incoming network traffic.

The main points I learned were:

* UFW can deny incoming connections by default.
* Specific ports can be allowed using firewall rules.
* A firewall rule can allow a service without changing the underlying network configuration.
* Removing the allow rule caused the service to become inaccessible again.
* Testing both the allowed and blocked states makes it possible to verify that the firewall rule is having the expected effect.
* Basic network troubleshooting is important when testing security controls, because the network connection itself needs to work before firewall behaviour can be meaningfully tested.

---

## Limitations

This was a small lab using a single HTTP service and a controlled VirtualBox Host-Only network.

It does not represent a full production firewall configuration.

The testing also focused on basic connectivity rather than more advanced firewall features or detailed traffic analysis.

---

## Future Improvements

If I continued developing this lab, I would like to:

* Test additional services and ports.
* Investigate UFW logging in more detail.
* Capture the traffic using Wireshark while changing firewall rules.
* Explore more specific firewall rules based on source addresses.
* Compare UFW configuration with lower-level Linux firewall concepts.

These are **future improvements rather than work completed in this lab**.
