# Wireguard VPN container

Container acting as Wireguard server or client.

WireGuard® is an extremely simple but fast and modern VPN. It aims to be faster, simpler, leaner, and more useful than IPsec, while avoiding the massive headache. It intends to be considerably more performant than OpenVPN. WireGuard is designed as a general purpose VPN for running on embedded interfaces and super computers alike, fit for many different circumstances.

Information related to Wireguard can be found at the official
[wireguard](https://www.wireguard.com/) webpage.

## Requirements

- Linux host with a recent kernel (> 5.6)
- Wireguard module installed
- Container runtime installed

## Setup

### Run as server

```sh
podman run --rm \
  --cap-add net_admin \
  --cap-add net_raw \
  -e SERVER_IP=x.x.x.x \
  -v /srv/wireguard:/etc/wireguard \
  -p 51820:51820/udp \
  --name wireguard \
  registry.opensuse.org/home/kukuk/container/container/opensuse/wireguard
```

The `-v /srv/wireguard:/etc/wireguard` is to store the configuration
permanently on disk, `/srv/wireguard` can be any existing local directory
where the wireguard configuration files should be stored.

### Add peers on the server

```sh
podman exec wireguard addpeer <ID>
```

where `<ID>` is an unique, alphanumeric only identifier for the peer.

The output will be the configuration file for the client and a *QR CODE*.

## Parameters

Container images are configured using parameters passed at runtime. The
following table contains the environment variables and their meaning to
configure wireguard at startup:

| Parameter | Function |
| :----: | --- |
| `-p 51820/udp` | Wireguard port. |
| `-e DEBUG=[0|1] | Enable debug mode for scripts, default off. |
| `-e NAT=[0|1] | Enable NAT using iptables, default off. Should be enabled on the server side, and only there. |
| `-e TZ=Europe/Berlin` | Specify a timezone to use e.g. Europe/Berlin. |
| `-e SERVER_IP=x.x.x.x` | External IP or domain name for container host. Used in server mode. If set to `auto`, the container will try to determine and set the external IP automatically with help of icanhazip.com |
| `-e SERVER_PORT=51820` | External port for container host. Used in server mode. |
| `-e PEERDNS=auto` | DNS server set in peer/client configs (can be set as `8.8.8.8`). Used in server mode. Defaults to `auto`, which uses wireguard docker host's DNS via included CoreDNS forward. |
| `-e SUBNET4=192.168.216.0` | Internal IPv4 subnet for the wireguard server and peers. |
| `-e ALLOWEDIPS=0.0.0.0/0` | The IPs/Ranges that the peers will be able to reach using the VPN connection. If not specified the default value is: '0.0.0.0/0, ::0/0', which will cause all traffic to route through the VPN. |
| `-v /etc/wireguard` | Contains all relevant wireguard configuration files. |
