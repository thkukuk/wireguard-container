# Wireguard VPN container

Small container acting as wireguard server or client.

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
podman run -d --rm \
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

#### Add peers on the server

```sh
podman exec wireguard addpeer <ID>
```

where `<ID>` is an unique, alphanumeric only identifier for the peer.

The output will be the configuration file for the client and a *QR CODE*.

### Run as peer

```sh
podman run -d --rm \
  --net=host \
  --cap-add net_admin \
  --cap-add net_raw \
  -v /srv/wireguard:/etc/wireguard \
  --name wireguard \
  registry.opensuse.org/home/kukuk/container/container/opensuse/wireguard
```

The `-v /srv/wireguard:/etc/wireguard` is to map the directory containing the
`wg0.conf` configuration file into `/etc/wireguard` of the container. This can
be any directory on the container host containing the configuration file.

The `--net=host` option makes the `wg0` interface visible and usable for all
processes on the container host. Without this option, the wireguard interface
is only visible and useable inside the container.

By default all traffic will be routed through the VPN as long as `ALLOWEDIPS` is
set to something different then `0.0.0.0`.

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
| `-e SERVER_IP=x.x.x.x` | External IP or domain name for container host. Used in server mode to create a new peer configuration. If not set or set to `auto`, the container will try to determine and set the external IP automatically with help of icanhazip.com |
| `-e SERVER_PORT=51820` | External port for container host. Used in server mode. |
| `-e PEERDNS=x.x.x.x` | DNS server set in peer/client config. Used when creating new peer configs on the server. If set to `auto` the  wireguard container host's IP is used. |
| `-e SUBNET4=192.168.216.0` | Internal IPv4 subnet for the wireguard server and peers. |
| `-e ALLOWEDIPS=0.0.0.0/0` | The IPs/Ranges that the peers will be able to reach using the VPN connection. If not specified the default value is: '0.0.0.0/0, ::0/0', which will cause all traffic to route through the VPN. |
| `-v /etc/wireguard` | Contains all relevant wireguard configuration files. |
