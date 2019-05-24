---
layout: post
title: Working Canon PIXMA G3010/G3110 @ macOS
date: 2018-09-11 13:55:00
published: true
comments: true
categories:
- Slice of life
tags:
- canon
- drivers
- macOS
- printers
author: desiderantes
---
The only way to make this work is by using it vía network, forget about
USB. Also, it only works as a printer, not as a scanner, blame Canon for
not even implementing proper Bonjour support as a multifunctional
printer. To make it work:

  - Download the drivers for the [Canon PIXMA
    G3000](https://www.usa.canon.com/internet/portal/us/home/support/details/printers/megatank-inkjet-printers/all-megatank-inkjet-printers/g3200/pixma-g3200?tab=drivers_downloads)
    (it says 3200 but it's the one for the 3000)
  - Download the [Canon Android
    APP](https://play.google.com/store/apps/details?id=jp.co.canon.bsd.ad.pixmaprint)
    and setup the printer to connect to your network.
  - Install the PIXMA G3000 drivers and setup the printer as a network
    printer.

Done. It shows up as a regular printer, and you can print from it (it
takes a little bit longer than printing from Windows), but it's just a
regular printer. The Android App might include some PPD files and/or
drivers to really make it work, but who knows? I made sure to ask
someone else to do the wireless setup for me so that I didn't have to
agree to any of the usually insane Terms & Conditions. This process
might work for Linux-based systems but I do not use any of them at work,
and I do not use printers at home.
