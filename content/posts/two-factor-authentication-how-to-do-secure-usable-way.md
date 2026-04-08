---
title: "Two-factor Authentication: How to Do It in a Secure and Usable Way"
date: 2020-03-22T00:00:00+00:00
lastmod: 2020-03-22T00:00:00+00:00
draft: false
tags: [security, 2fa, authentication]
---

![Two-factor authentication illustration](/images/two-factor-authentication-how-to-do-secure-usable-way/free-vector-online-identification-illustration.svg)

If you enable two-factor authentication on every service where it's possible — and you use many of those services every day — you've probably felt the friction of manually entering 2FA codes. The usual process: open your authenticator app (Authy, Google Authenticator, etc.) on your phone, find the code, type it in. Do this several times a day and it becomes cumbersome. But there's a better way.

You probably already know you should use a password manager for your service-specific passwords — long, random, unique passwords for each service, with only one master password to remember. What most people don't know is that your password manager can likely handle 2FA as well.

Here's how I do it. I use two applications to store my 2FA codes: [Authy](https://authy.com/) on my phone and [1Password](https://1password.com/) on my computer. When setting up 2FA on a service, I scan the QR code into both Authy and 1Password, and also save the 2FA backup codes to 1Password.

* **Authy** gives me a backup of my 2FA codes if something goes wrong with 1Password
* **1Password** gives me ease of use on my computer — when it auto-fills login credentials, it copies the 2FA code to the clipboard so I can paste it immediately

Simple as that. Instructions for setting this up in 1Password: [support.1password.com/one-time-passwords](https://support.1password.com/one-time-passwords/#save-your-qr-code)

Happy account securing!

*Header image by [Vecteezy](https://www.vecteezy.com/free-vector/theft)*
