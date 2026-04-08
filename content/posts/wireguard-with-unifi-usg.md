---
title: "WireGuard with Unifi Security Gateway Pro 4"
date: 2020-03-23T00:00:00+00:00
lastmod: 2020-08-27T00:00:00+00:00
draft: false
tags: [unifi, wireguard, usg, vpn]
---

![WireGuard logo](/images/wireguard-with-unifi-usg/wireguard-1.svg)

As part of my home network I have setup VPN connectivity so that I can access my stuff also when I'm not at home. Unifi Security Gateway offers PPTP and L2TP VPN servers out of the box but there are better alternatives available like [WireGuard](https://www.wireguard.com/) and OpenVPN. As with everything I wanted to learn new stuff so I chose WireGuard for this task. "WireGuard is an extremely simple yet fast and modern VPN" that "aims to be as easy to configure and deploy as SSH." A VPN connection is made simply by exchanging very simple public keys — exactly like exchanging SSH keys — and all the rest is transparently handled by WireGuard. Unfortunately the information on how to get WireGuard up & running and make the installation persistent after USG upgrades could not be found from one single place. This is why I decided to gather the data under this one post.

## Installing WireGuard

1. Login to your USG via SSH.

2. Download the latest `ugw4` package from [wireguard-vyatta-ubnt releases](https://github.com/WireGuard/wireguard-vyatta-ubnt/releases) and install it on your USG:

```bash
curl -fLSs -o /tmp/ugw4-v1-v1.0.20200729-v1.0.20200513.deb \
  https://github.com/WireGuard/wireguard-vyatta-ubnt/releases/download/1.0.20200729-1/ugw4-v1-v1.0.20200729-v1.0.20200513.deb
dpkg -i /tmp/ugw4-v1-v1.0.20200729-v1.0.20200513.deb
```

3. Create private and public keys. Copy the public key somewhere as you will need it when adding a peer in your device's WireGuard settings:

```bash
cd /config/auth
umask 077
mkdir wireguard
cd wireguard
wg genkey > wg_private.key
wg pubkey < wg_private.key > wg_public.key
```

4. Copy the example `config.gateway.json` to `/srv/unifi/data/sites/default` (CloudKey) or `/var/lib/unifi/data/sites/default` (Raspberry Pi etc.), using the site name you're running instead of `default`. Change `YOUR-DEVICE-PUBLIC-KEY` to a public key generated on your WireGuard client device. Then through the Controller Web UI navigate to Devices, click the USG row, and in the Properties window go to Config > Manage Device and click Provision. Note that the mask associated with `allowed-ips` is not a netmask:

```json
{
    "firewall": {
        "group": {
            "network-group": {
                "remote_user_vpn_network": {
                    "description": "Remote User VPN subnets",
                    "network": [
                        "10.2.1.0/24"
                    ]
                }
            }
        }
    },
    "interfaces": {
        "wireguard": {
            "wg0": {
                "address": [
                    "10.2.1.1/24"
                ],
                "firewall": {
                    "in": {
                        "name": "LAN_IN"
                    },
                    "local": {
                        "name": "LAN_LOCAL"
                    },
                    "out": {
                        "name": "LAN_OUT"
                    }
                },
                "listen-port": "51820",
                "mtu": "1352",
                "peer": [{
                    "YOUR-DEVICE-PUBLIC-KEY": {
                        "allowed-ips": [
                            "10.2.1.5/32"
                        ],
                        "persistent-keepalive": 25
                    }
                }],
                "private-key": "/config/auth/wireguard/wg_private.key",
                "route-allowed-ips": "true"
            }
        }
    }
}
```

5. To allow remote access, navigate to Settings > Routing & Firewall > Firewall > WAN LOCAL and create a new rule to accept UDP traffic to port 51820. You need to create a Port Group under Destination and add port 51820 via that. Remember to make the changes under the same site where you placed the config file.

![WireGuard firewall WAN LOCAL rule](/images/wireguard-with-unifi-usg/wireguard-firewall-wan-local.jpg)

6. Install WireGuard on your device and configure it with the following. With this configuration all traffic will be sent through the VPN server. Full config reference: [wireguard-docs](https://github.com/pirate/wireguard-docs#Config-Reference).

```ini
[Interface]
PrivateKey = YOUR-DEVICES-PRIVATE-KEY
Address = 10.2.1.5/32
DNS = 1.1.1.1

[Peer]
PublicKey = YOUR-SERVERS-PUBLIC-KEY
AllowedIPs = 0.0.0.0/0
Endpoint = YOUR-SERVERS-PUBLIC-IP:51820
PersistentKeepalive = 21
```

7. You can generate a QR code to easily configure your phone by running:

```bash
qrencode -t ansiutf8 <path_to_config_file>
```

8. You should now be able to make a VPN connection with WireGuard to your USG.

## Persisting Changes

All CLI-made configurations and installed applications will be overwritten when you upgrade or reboot your USG. Fortunately there is a way to persist these changes. The folks at [Helm Rock Consulting](https://helmrock.com/) have implemented `install-edgeos-packages` to solve exactly this problem.

1. Install `install-edgeos-packages`:

```bash
curl -O https://raw.githubusercontent.com/britannic/install-edgeos-packages/master/install-pkgs
sudo install -o root -g root -m 0755 install-pkgs /config/scripts/post-config.d/install-pkgs
```

2. Add the WireGuard package so it gets reinstalled after upgrade or reboot:

```bash
sudo mkdir -p /config/data/install-packages
cd /config/data/install-packages
curl -fLSs -O https://github.com/WireGuard/wireguard-vyatta-ubnt/releases/download/1.0.20200729-1/ugw4-v1-v1.0.20200729-v1.0.20200513.deb
```

3. Export and copy `config.gateway.json` to the Controller host. This ensures CLI changes are provisioned back to the USG after reboot or upgrade. To get your config, run via SSH on the USG:

```bash
mca-ctrl -t dump-cfg > config.gateway.json
```

After these steps you have a fully functional WireGuard server installation that persists through reboots and upgrades.

**Sources:** [pamolloy's gist](https://gist.github.com/pamolloy/059c552b814b0dddfcdc0cec2bbe5872) and [install-edgeos-packages docs](https://britannic.github.io/install-edgeos-packages/)
