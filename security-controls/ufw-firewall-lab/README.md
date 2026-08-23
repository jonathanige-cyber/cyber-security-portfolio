# UFW Firewall Security Control Lab

## Objective

Configure and test a host-based firewall in Kali Linux to demonstrate how firewall rules can prevent unauthorised network access.

## T Level Knowledge

This practical links to:

- Network security
- Security controls
- Firewall configuration
- Access control
- Network monitoring and testing
- Security testing and verification

## Environment

- Kali Linux
- VirtualBox
- UFW (Uncomplicated Firewall)
- Python HTTP server
- Windows host machine
- Host-only network: `192.168.56.0/24`

## Security Control

**Control:** Host-based firewall  
**Type:** Preventive

A firewall can restrict network traffic based on rules such as source, destination, protocol and port.

## Initial Configuration

UFW was installed and initially inactive.

The firewall was enabled:

```bash
sudo ufw enable
