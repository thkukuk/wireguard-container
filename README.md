# Wireguard VPN container

Small container acting as wireguard server or client.

WireGuard® is an extremely simple but fast and modern VPN. It aims to be faster, simpler, leaner, and more useful than IPsec, while avoiding the massive headache. It intends to be considerably more performant than OpenVPN. WireGuard is designed as a general purpose VPN for running on embedded interfaces and super computers alike, fit for many different circumstances.

Information related to Wireguard can be found at the official
[wireguard](https://www.wireguard.com/) webpage.

## Requirements

- Linux host with a recent kernel (> 5.6)
- Wireguard module installed
  - If SELinux is in use, the wireguard kernel module must be loaded before the container is started, e.g. `echo wireguard > /etc/modules-load.d/wireguard.conf` will do that at every boot.
- Container runtime installed

## Setup

### Run as server

```sh
podman run -d --rm \
  --cap-add=net_admin \
  --cap-add=net_raw \
  --cap-add=sys_module \
  --sysctl=net.ipv4.conf.all.src_valid_mark=1 \
  --sysctl=net.ipv4.ip_forward=1 \
  -e SERVER_IP=x.x.x.x \
  -v /srv/wireguard:/etc/wireguard:Z \
  -p 51820:51820/udp \
  --name wireguard \
  registry.opensuse.org/home/kukuk/container/wireguard:latest
```

The `-v /srv/wireguard:/etc/wireguard:Z` is to store the configuration
permanently on disk, `/srv/wireguard` can be any existing local directory
where the wireguard configuration files should be stored.

The provided environment variables are used to create the `wg0.conf` for the server at the first start and the peers with the `addpeer` command. If the config file for the server or peer already exists, changing the environment variables will have no effect.

#### Add peers on the server

```sh
podman exec wireguard addpeer <ID>
```

where `<ID>` is an unique, alphanumeric only identifier for the peer.

The output will be the configuration file for the client and a *QR CODE*.

#### Show peer configuration

The following command can be used to display the configuration files and QR codes of peers again:

```sh
podman exec wireguard showpeer <ID> ...
```

where `<ID>` is the unique, alphanumeric only identifier for the peer.

#### Remove peer from the server

```sh
podman exec wireguard delpeer <ID>
```

where `<ID>` is an unique, alphanumeric only identifier for the peer.

This will remove the peer from the active wireguard setup and delete all
configuration files on the server.

### Run as peer

Get the peer config from the server (`podman exec wireguard showpeer <ID>`) and
store them as `/etc/wireguard/wg0.conf`.

**IMPORTANT**: The DNS option does not work and must be removed in advance from
`wg0.conf` and manually added to `/etc/resolv.conf` on the host if wireguard is
running as a container on the peer. Wireguard cannot change the `/etc/resolv.conf`
file inside the container and applications outside the container will not see the
new DNS entry, as it is only visible inside the container.

```sh
podman run -d --rm \
  --net=host \
  --cap-add=net_admin \
  --cap-add=net_raw \
  -v /etc/wireguard:/etc/wireguard:Z \
  --name wireguard \
  registry.opensuse.org/home/kukuk/container/wireguard:latest
```

The `-v /etc/wireguard:/etc/wireguard:Z` is to map the directory containing the
`wg0.conf` configuration file into `/etc/wireguard` of the container. This can
be any directory on the container host containing the configuration file.

The `--net=host` option makes the `wg0` interface visible and usable for all
processes on the container host. Without this option, the wireguard interface
is only visible and useable inside the container.

By default all traffic will be routed through the VPN as long as `ALLOWEDIPS`
is not set to something different then `0.0.0.0`.

## Parameters

Container images are configured using parameters passed at runtime. The
following table contains the environment variables and their meaning to
configure wireguard at startup:

| Parameter | Function |
| :----: | --- |
| `-p 51820/udp` | Wireguard port. |
| `-e DEBUG=[0/1]` | Enable debug mode for scripts, default off. |
| `-e TZ=Europe/Berlin` | Specify a timezone to use e.g. Europe/Berlin. |
| `-e SERVER_IP=x.x.x.x` | External IP or domain name for container host. Used in server mode to create a new peer configuration. If not set or set to `auto`, the container will try to determine and set the external IP automatically with help of icanhazip.com |
| `-e SERVER_PORT=51820` | External port for container host. Used in server mode. |
| `-e PEERDNS=x.x.x.x` | DNS server set in peer/client config. Used when creating new peer configs on the server. If set to `auto` the  wireguard container host's IP is used. |
| `-e SUBNET4=192.168.216.0` | Internal IPv4 subnet for the wireguard server and peers. |
| `-e ALLOWEDIPS=0.0.0.0/0` | The IPs/Ranges that the peers will be able to reach using the VPN connection. If not specified the default value is: '0.0.0.0/0, ::0/0', which will cause all traffic to route through the VPN. |
| `-v /etc/wireguard` | Contains all relevant wireguard configuration files. |
| `-e INTERFACE=eth0` | Define a different interface for `iptables` rules inside the container. |
| `-e EXPORT_METRICS=[0/1]` | Export metrics for prometheus. |
| `-p 9586:9586` | Port to export prometheus metrics. |

## Additonal informations

### podman quadlet

An example podman quadlet file (`/etc/containers/systemd/wireguard.container`) for the wireguard server:

```
[Unit]
Description=Wireguard Server Container
After=local-fs.target

[Container]
ContainerName=wireguard
Image=registry.opensuse.org/home/kukuk/container/wireguard:latest
Pull=newer
#Exec=
AddCapability=NET_ADMIN
AddCapability=NET_RAW
AddCapability=SYS_MODULE
Sysctl=net.ipv4.conf.all.src_valid_mark=1
Sysctl=net.ipv4.ip_forward=1
Volume=/srv/wireguard:/etc/wireguard:Z
# Port for wireguard tunnel
PublishPort=51820:51820/udp
Environment=SERVER_IP=gw.example.com
Environment=SUBNET4=10.80.12.0
Environment=PEERDNS=10.80.12.1
Environment=ALLOWEDIPS=10.80.12.0/24
# Export metrics for prometheus
PublishPort=9586:9586
Environment=EXPORT_METRICS=1

[Service]
Restart=on-failure

[Install]
# Start by default on boot
WantedBy=multi-user.target default.target
```
