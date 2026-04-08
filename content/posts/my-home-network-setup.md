---
title: "My Home Network Setup"
date: 2020-03-23T00:00:00+00:00
lastmod: 2020-03-23T00:00:00+00:00
draft: false
tags: [network, unifi, pi-hole, kubernetes, vlan]
---

![Server cables](/images/my-home-network-setup/server-cables.jpg)

Previously I had a Ubiquiti Edgerouter Lite, two managed Netgear switches and a [Unifi AP-SHD WiFi](https://unifi-shd.ui.com/). When my Edgerouter Lite suddenly died and there was no warranty left, I thought I could redo my home network and learn something new while doing it. I had some principles I wanted to fulfil:

* The setup needed to be robust
* The setup needed to offer great configurability
* I wanted the possibility to isolate different devices into different VLANs
* I wanted PoE capability

For this task I chose the [Unifi product family](https://unifi-network.ui.com/) from [Ubiquiti](https://www.ui.com/) because they combine quality hardware with great configurability, and I had heard good things about them. I decided the Unifi AP-SHD WiFi was good enough to stay, but the rest I wanted to replace. I got the following gear:

* [Unifi Security Gateway Pro 4](https://www.ui.com/unifi-routing/unifi-security-gateway-pro-4/)
* [Unifi US-24-250W Managed PoE Switch](https://www.ui.com/unifi-switching/unifi-switch-poe/)
* [Unifi Cloud Key Gen2 Plus](https://unifi-protect.ui.com/cloud-key-gen2)
* [Unifi Cloud Key G2 Rack Mount](https://store.ui.com/products/cloud-key-g2-rack-mount-accessory)

Physically I have set up my devices so that both the USG and US-24-250W switch are behind a UPS, and the Cloud Key & AP-SHD are powered via PoE from the switch. All the devices except the WiFi box are sitting in a small closet while I wait for my storage room renovation to finish — after that I'll assemble a proper rack. So the final cabling setup will have to wait.

Network-wise I have set up the following [VLANs](https://en.wikipedia.org/wiki/Virtual_LAN):

* **Management VLAN** — all network gear
* **LAN VLAN** — computers, tablets, and phones
* **IoT VLAN** — media boxes, TVs, etc.
* **Playground VLAN** — home servers

IoT devices cannot connect to internal networks, but LAN devices can reach IoT devices. This way I can stream from my phone to my Apple TVs. I've also set up [Pi-hole](https://pi-hole.net/) for DNS-level ad blocking — no browser extensions needed, and it limits how much data IoT devices send to the internet. My Pi-hole runs on a Kubernetes cluster across three Intel NUCs. I've set up [MetalLB](https://metallb.universe.tf/) with BGP for load balancing, giving actual fault tolerance to the cluster. And for remote access I've set up [WireGuard VPN](https://www.wireguard.com/) on the USG.

My setup is admittedly more than strictly needed for a home network, but I've been wanting to learn new things — hence the three-node Kubernetes cluster. I'll publish more detailed posts on the Pi-hole and WireGuard setups in the future.
