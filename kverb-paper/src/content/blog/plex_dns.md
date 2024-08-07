---
author: Ken V
title: Plex Isn't Matching Any New Movies
pubDatetime: 2022-03-01T19:11:32-04:00
draft: false
featured: false
description: How I resolved an issue with Plex not matching any newly added items.
tags: 
 - technical
 - notes
---

After a server restart and <code>pacman -Syu</code> , plex isn’t matching any new added movie files.
Permissions are good and names are fine.
Manually trying to search the scanners in the UI instantly fails.
Nothing informative in the logs. 

Trying to update with `yay -Sy plex-media-server` and restarting doesn't help.

Turns out it was some kind of DNS problem.
DNS resolving was working fine in a bash shell and other programs but for some unknown reason, plex wasn’t able to resolve anything.
Works after disabling tailscale and resetting `/etc/resolv.conf` to non-tailscale values.
