---
author: Ken V
title: "Arch Linux New Install Bootstrap Steps"
pubDatetime: 2022-06-11T19:11:32-04:00
draft: false
featured: true
description: First steps to take when bootstrapping an arch linux container in proxmox
tags:
 - technical
---

Although I am continually delighted by arch linux, one thing I find frustrating is the 'bootstrap' process when using arch in something like an LXC / PVE container. 
It's rather annoying that these somewhat convoluted steps must be taken to do a simple system update or package install.
I'm documenting here so that I and maybe others have a convenient place to reference it.

{{< alert >}}
Assumes you are using the basic arch linux image in a proxmox virtualization environment 
{{< /alert >}}

- create an arch LXC container on proxmox.
  - (optional) include your ssh public key during creation.
- open the shell or console in PVE and enable sshd: `systemctl enable --now sshd`
- if you did not add your public key for ssh, then you'll also need to enable password authentication for root in `/etc/ssh/sshd_config`

Once you can ssh in, (or continue in the PVE console, yuck), you will need to enable the pacman mirrorlist from [here](https://archlinux.org/mirrorlist/all/https/) (uncomment the US section at the bottom) and copy it to <code>/etc/pacman.d/mirrorlist</code>

You can then run the following commands:

```bash
pacman-key --init
pacman-key --populate archlinux
pacman -Syy --noconfirm archlinux-keyring
```
  
You are now ready to  update with `pacman -Syyu --noconfirm`

- install python (required for ansible) `pacman -S python`

 Now you can run ansible via ssh, but be sure to include the new container in your ansible inventory.

## more info
- see [the arch wiki](https://wiki.archlinux.org/title/Pacman/Package_signing#Initializing_the_keyring) for more about pacman-key

