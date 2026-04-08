---
title: "Upgrading Firmware for Netgear GS105Ev2 Switch on Mac OS"
date: 2017-04-08T00:00:00+00:00
lastmod: 2017-04-08T00:00:00+00:00
draft: false
tags: [netgear, firmware, mac]
---

![Network cables](/images/upgrading-firmware-for-netgear-gs105ev2-switch-on-mac-os/network-cable-ethernet-computer-159304.jpeg)

I recently got a Netgear GS105Ev2 and GS108Ev3 managed switch and wanted to update their firmware. The GS108Ev3 updated easily through its web interface, but the GS105Ev2 required a different approach using TFTP. Here's how to do it on Mac OS.

1. Download the latest firmware from the Netgear support site
2. Extract the downloaded .zip file
3. Download the [MacTFTP Client](https://www.mactftpclient.com/)
4. Connect your Mac and the GS105Ev2 switch directly to each other with a wired connection
5. Remove all other network cables from the GS105Ev2 switch
6. Power off and power on the GS105Ev2 switch
7. Configure your Mac to use a static IP address of `192.168.0.1` with netmask `255.255.255.0`
8. Connect to the GS105Ev2 switch via web browser at `http://192.168.0.239` with password `password`
9. Head to Maintenance → Firmware Upgrade and press "Enter Loader mode"
10. Change the URL manually to `http://192.168.0.239/index.htm`
11. Fire up the MacTFTP Client downloaded earlier
12. Select "Send", use `192.168.0.239` as the address and `password` as the password
13. Select the firmware file using the "File..." button and click "Start"
14. Wait for a minute for the upload to complete, then for the switch to update and reboot
15. Connect to `http://192.168.0.239` — it will ask you to upload the `-VB.bin` bootloader update file
