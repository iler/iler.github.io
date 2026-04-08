---
title: "Dynamic DNS (dyndns) with Namecheap on Unifi Security Gateway Pro 4"
date: 2020-03-27T00:00:00+00:00
lastmod: 2020-03-27T00:00:00+00:00
draft: false
tags: [namecheap, dyndns, usg, unifi, vpn]
---

Home network users commonly deal with dynamic IP addresses that change unpredictably. This can be troublesome when using a VPN, as the IP can change at any time. Dynamic DNS solves this by running a daemon that updates DNS records whenever the IP address changes, creating the illusion of a static address.

## Namecheap Configuration

1. Log in to your Namecheap account
2. Select "Manage" for your desired domain
3. Navigate to Advanced DNS
4. Activate the Dynamic DNS service and save the password
5. Add an A+Dynamic DNS Record type
6. Use `@` for naked domains or specify a subdomain
7. Set `127.0.0.1` as the initial IP address (the daemon will replace this)
8. Save changes

## USG Controller Configuration

1. Log in to the USG controller
2. Go to Settings > Services > Dynamic DNS
3. Click "Create new dynamic DNS"
4. Enter these details:
   - **WAN Interface**: Select your active WAN interface
   - **Service**: Choose Namecheap
   - **Hostname**: Use `@` or your configured subdomain
   - **Username**: Your domain name
   - **Password**: The Dynamic DNS service password from Namecheap
   - **Server**: `dynamicdns.park-your-domain.com`
5. Click Save

That's it — the daemon will now keep your DNS record updated whenever your public IP changes.
