# AdGuard Home for Synology Container Manager

This guide provides instructions on how to set up AdGuard Home in a Docker container on a Synology DS723+ with DSM 7.2 using Synology Container Manager. The setup includes macvlan networking to allow AdGuard Home to have its own dedicated IP address and supports IPv6 functionality.

Testet with Synology DS723+ with DSM 7.2 and Fritzbox 6591 Cable

## Features

- Dedicated IP via macvlan
- Full IPv6 support
- Efficient ad blocking with AdGuard Home
- Integration with Synology DSM 7.2 and Docker

## Install
Modifying the Docker Configuration via SSH
The file /volume1/@appconf/ContainerManager/dockerd.json contains the settings for the Docker daemon. To enable IPv6, add the following entries:
```
sudo nano /volume1/@appconf/ContainerManager/dockerd.json
```

```json
{
  "ipv6": true,
  "fixed-cidr-v6": "fd00::/80"
}
```
### Explanation of the Parameters:
  - ```"ipv6": true``` → Enables IPv6 support in the Docker daemon
  - ```"fixed-cidr-v6": "fd00::/80"``` → Assigns a fixed internal IPv6 subnet (fd00::/80) from which Docker can allocate IPv6 addresses to containers.
    
### Why Is This Necessary?

- Without this change, Docker does not assign IPv6 addresses to containers.
- AdGuardHome and other services cannot handle IPv6 queries without proper IPv6 support.
- IPv6 networking (e.g., macvlan with IPv6) only works if Docker is IPv6-enabled.

After modifying the configuration, you must restart the Container Manager for the changes to take effect:
```
sudo synopkg restart ContainerManager
```

## compose.yaml
```xaml
version: '3'

services:
  adguardhome:
    image: adguard/adguardhome
    container_name: adguardhome-server
    hostname: adguard
    restart: always
    networks:
      adguard-macvlan:
        ipv4_address: 192.168.178.245
        ipv6_address: 2001:db8:abcd:1234::245  # Random IPv6 static ip adress for documentation, replace with your real ip adress.
    security_opt:
      - no-new-privileges:true
    cap_add:
      - NET_ADMIN
    ports:
      - "53:53/tcp" # Unencrypted DNS
      - "53:53/udp" # Unencrypted DNS
      - "67:67/udp" # DHCP
      - "68:68/udp" # DHCP
      - "80:80/tcp" # Admin web interface and DNS over HTTPS
      - "443:443/tcp" # Admin web interface and DNS over HTTPS
      - "443:443/udp" # Admin web interface and DNS over HTTPS
      - "3000:3000/tcp" # Initial setup
      - "853:853/tcp" # DNS over TLS
      - "853:853/udp" # DNS over QUIC
      - "784:784/udp" # DNS over QUIC
      - "8853:8853/udp" # DNS over QUIC
      - "5443:5443/tcp" # DNSCrypt
      - "5443:5443/udp" # DNSCrypt
    volumes:
      - "/volume1/docker/adguard/data:/opt/adguardhome/work/data"
      - "/volume1/docker/adguard/config:/opt/adguardhome/conf"
      - "/etc/localtime:/etc/localtime:ro"

networks:
  adguard-macvlan:
    name: adguard-macvlan
    enable_ipv6: true
    driver: macvlan
    driver_opts:
      parent: eth0
    ipam:
      config:
        - subnet: "192.168.178.0/24"
          ip_range: "192.168.178.254/24"
          gateway: "192.168.178.1"
        - subnet: "2001:db8:abcd:1234::/64"  # Random IPv6 subnet for documentation, replace with your real subnet.
          gateway: "2001:db8:abcd:1234::1"   # Random IPv6 gateway, change to match your router configuration.
```

## Support & Discussions
<a href="https://github.com/riggi89/adguard-docker/issues">GitHub Issues</a>
