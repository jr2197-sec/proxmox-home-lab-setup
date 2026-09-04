# Proxmox VE Home Lab Setup

This repository documents my first Proxmox VE home-lab installation and the networking troubleshooting I performed to get the server online and accessible from my Windows PC.

## Project Goals

- Install and boot Proxmox VE on a mini PC
- Configure Ethernet networking and the `vmbr0` Linux bridge
- Troubleshoot an incorrect IP/gateway configuration
- Verify LAN and Internet connectivity
- Access the Proxmox VE web GUI remotely from Windows
- Document the lab for my IT/cybersecurity portfolio

## Environment

- Mini PC running Proxmox VE
- Windows PC for remote administration
- Wired Ethernet connection
- Physical Ethernet interface: `enp2s0`
- Proxmox Linux bridge: `vmbr0`
- Wireless interface detected during troubleshooting: `wlp3s0`

## Installation and Troubleshooting

### 1. Booted Proxmox VE

After installing Proxmox VE, I logged into the local console as `root`. Proxmox provides a browser-based management interface over HTTPS on TCP port `8006`.

### 2. Inspected the network configuration

I opened the Debian/Proxmox network configuration with:

```bash
nano /etc/network/interfaces
```

I identified `enp2s0` as the physical Ethernet interface and `vmbr0` as the Linux bridge used by Proxmox.

### 3. Diagnosed the original static configuration

The original static network settings did not match the active home LAN. This resulted in errors such as:

```text
Network is unreachable
```

I inspected the configuration and interface state with:

```bash
cat /etc/network/interfaces
ip -4 addr show vmbr0
```

I also tested Internet connectivity with:

```bash
ping -c 4 8.8.8.8
```

### 4. Obtained a valid DHCP lease

During troubleshooting I requested a DHCP lease for the bridge:

```bash
dhcpcd vmbr0
```

The router assigned the Proxmox server a valid address on the home LAN. I confirmed it with:

```bash
ip -4 addr show vmbr0
```

and tested Internet connectivity again:

```bash
ping -c 4 8.8.8.8
```

Receiving replies with `0% packet loss` confirmed that the server could reach the Internet.

### 5. Configured `vmbr0` for DHCP

For the working lab configuration, I changed `vmbr0` to obtain its IPv4 configuration through DHCP:

```text
auto lo
iface lo inet loopback

iface enp2s0 inet manual

auto vmbr0
iface vmbr0 inet dhcp
        bridge-ports enp2s0
        bridge-stp off
        bridge-fd 0

iface nic1 inet manual

source /etc/network/interfaces.d/*
```

I then reloaded the networking configuration:

```bash
ifreload -a
```

### 6. Verified connectivity

I checked the bridge address:

```bash
ip -4 addr show vmbr0
```

Then tested external connectivity:

```bash
ping -c 4 8.8.8.8
```

This confirmed that the Proxmox host had a valid LAN address and Internet connectivity.

### 7. Accessed the Proxmox GUI from Windows

With the Windows PC and Proxmox host connected to the same LAN, I opened a browser and navigated to:

```text
https://<PROXMOX-IP>:8006/
```

After accepting the local certificate warning, the **Proxmox VE Login** page loaded successfully.

Login configuration:

- Username: `root`
- Realm: `Linux PAM standard authentication`
- Password: root password created during installation

At this point I had successfully moved from administering Proxmox locally at the console to managing it remotely through the web GUI.

## Commands Practiced

```bash
nano /etc/network/interfaces
cat /etc/network/interfaces
ifreload -a
ip -4 addr show vmbr0
ip link show wlp3s0
dhcpcd vmbr0
ping -c 4 8.8.8.8
```

## Problems Encountered and What I Learned

### Incorrect IP configuration

The original static IP/gateway settings did not match the active LAN, preventing normal network communication.

**Solution:** I used DHCP to obtain a valid configuration from the router and verified the resulting address and connectivity.

### Physical link vs. IP connectivity

The Ethernet link could be physically active while the server still lacked working Layer 3 connectivity. This reinforced the difference between having a live Ethernet connection and having a correct IP address, subnet, route, and gateway.

### Proxmox enterprise repository errors

While testing package installation, `apt` returned `401 Unauthorized` errors from Proxmox enterprise repositories. This was separate from the network problem: the server already had Internet connectivity, but the enterprise repository requires the appropriate subscription access.

### Wi-Fi investigation

The wireless interface `wlp3s0` was detected while troubleshooting. I ultimately kept the Proxmox host on wired Ethernet because a stable wired connection is preferable for this home-lab virtualization server when Ethernet is available.

## Networking Concepts Reinforced

- IPv4 addressing and subnetting
- Default gateways
- DHCP
- Linux network interfaces
- Linux bridges
- Physical vs. logical network connectivity
- ICMP/ping troubleshooting
- HTTPS management interfaces
- TCP port `8006`

## Result

**Successful.** The Proxmox VE host is online over Ethernet, receives a valid address from the home LAN, reaches the Internet, and can be remotely administered from my Windows PC through the Proxmox web interface.

## Skills Demonstrated

- Proxmox VE installation and administration
- Linux command-line usage
- Debian network configuration
- IPv4 and DHCP troubleshooting
- Linux bridge configuration
- Ethernet troubleshooting
- Connectivity testing
- Remote server administration
- Technical documentation

## Screenshots

Screenshots from the installation and troubleshooting process will be stored in the `images/` directory. Sensitive information such as passwords, credentials, and API keys should never be included in public screenshots.

---

This is a hands-on learning project documenting my progress building practical Linux, networking, virtualization, and cybersecurity skills.