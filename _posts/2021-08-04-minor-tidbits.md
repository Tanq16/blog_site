---
title: Minor TidBits
date: 2021-08-04 12:00:00 +0500
categories: [Other Stuff]
tags: [scripts,tidbits,linux,cli]
---

# Windows Change DNS Script - PowerShell

```
PowerShell.exe -NoProfile -WindowStyle Hidden -Command "& {Start-Process PowerShell.exe -WindowStyle Hidden -ArgumentList '-NoProfile -ExecutionPolicy Bypass -Command "Set-DnsClientServerAddress -InterfaceAlias Wi-Fi -ServerAddresses 1.1.1.1"' -Verb RunAs}"
```

# Prepare ubuntu VM for ssh only access

```bash
sudo apt remove thunderbird firefox libreoffice-core libreoffice-base-core \
	gnome-mahjongg gnome-calendar gnome-mines gnome-bluetooth gnome-screenshot \
	gnome-todo gnome-todo-common gnome-video-effects gnome-startup-applications \
	gnome-sudoku gnome-calculator aisleriot rhythmbox gedit transmission \
	gnome-power-manager gnome-themes-extra gnome-themes-extra-data bluez evince \	emacs emacs-common evince-common nautilus nautilus-sendto nautilus-data \
	nautilus-extension-gnome-terminal transmission-gtk transmission-common

sudo apt install -y curl vim zsh wget git strace ltrace gdb python3-pip nmap ncat

# visit github.com/tanq16/cli-productivity-suite
```

# Virtual Box network adapters

To configure multiple network adapters in Virtual Box if one of them turn down when the other is functional, add the following to `/etc/network/interfaces`

```
allow-hotplug
iface inet dhcp
```

# Shell Print

```bash
printf "%0.sA" {1..10}
```

prints `AAAAAAAAAA`
