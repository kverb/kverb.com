---
author: Ken V
title: Geary
pubDatetime: 2022-06-11T19:11:32-04:00
draft: false
featured: false
description: A short review of the Geary E-mail client on Linux
tags:
 - technical
---

Finally, I've found a relatively simple and functional GUI e-mail client for Linux - [Geary](https://gitlab.gnome.org/GNOME/geary/). 

Although it is built for the Gnome desktop environment, it works fine in SwayWM, runs in wayland, even with some nice transition animations.
It also (and i'm totally guessing here)  seems to obey system-wide GTK theme configuration.
I'm guessing because I never actually configured this myself; Garuda Linux on Sway seems to have shipped with a [nord-esque](https://www.nordtheme.com/) theme out of the box.

It has a basic set of GUI email client features, but the most important for me is "single-key keyboard shortcuts", similar to how one might use mutt or the Gmail web interface.
It's unfortunate how few email clients support this, but the vi-like navigation, including 'e' for archive has become so ingrained to my muscle memory that anything that ships without this is a deal-breaker for me.

Another nice feature is jamming 'j' and 'k' to navigate through emails is FAST. 


## Proton Mail Bridge integration

I've been using [Proton Mail](https://protonmail.com) with a custom domain for a few years, and while the mobile app is ... fine, the webmail has never worked well for me, even with the "new and improved" flavor. 
Interestingly, Proton mail doesn't support IMAP directly.
You must first run a local "bridge" daemon client, which presumably handles the encrypt/decrypt transport to proton servers, then makes your email locally available via IMAP.
It seems all a bit convoluted at first, but is overall quite a brilliant solution. And I was pleasantly surprised at easy the bridge app was to use and configure with Geary.
Similarly to Gmail "application passwords", you can generate a new app-specific password for any of the mail aliases you have in proton (i think you have to enable the "splt addresses" feature in the account settings).
In Geary, point your email account's IMAP server to 127.0.0.1 and the correct port, and voila. An added benefit, for me, is that because I use a "blackhole@" account, and allow <anything>@ to forward into that account, I can now have a separate inbox for all those ham emails.
Anytime I have to sign up for a service, I'll usually use a specific email alias for that service, and it will end up in the blackhole@ inbox.
This is great because it typically just drops all the crappy marketing emails and newsletters there, while keeping my personal ken@ inbox cleaner.

{{< alert >}}
NOTE: On proton mobile, it all just ends up in one inbox.
{{< /alert >}}

## Flaws but not deal-breakers

 - Possibly a sway limitation, but I can't seem to be able to increase the font size in the message view list.
 - it's simple on features, and i probably wouldn't sync a lifetime of mail to my local system, which limits the search capabilities.
 - HTML formatting is fine, but feel different from the gmail webmail interface.
 - There's no shortcut to navigate between messages in a conversation. It seems like you have to use <Tab> and <Enter>. It's a bit clunky.
