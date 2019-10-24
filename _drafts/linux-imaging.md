---
title: "Linux Imaging Center"
author: bkelley
layout: post
date: 2019-- ::00 -0600
categories: posts
image: /assets/posts/.png
description: >
  Creating Linux images
---

## Imaging A Fresh PC

### Logic Supply PC

1. Plug in a live Fedora USB [Or create a new one](/procedures/live-usb.html), into a fresh machine while it is powered off
2. Hold delete key and power the pc on
3. Use the right arrow key to select the "Save and Exit" tab
4. Under the boot override section highlight you're thumb drive using the up or down arrow key
5. Hit enter.
6. Select `Start fedora live` from the boot menu (It should be the option one up from what is selected by default, use the up arrow to select it)
7. Once it has booted close the window that asks whether you would like to try or install Fedora by hitting the x button with the mouse
8. Plug in a USB with the image on it (Image can be obtained at `Rdata Overflow/R&D Ivory Tower/VersaBuilt/VBXC-Images`)
9. Open a terminal (press the windows key, type `terminal` and hit enter)
10. Switch to the root user by typing `su` and hitting enter
11. Type `gzip -dc /run/media/liveuser/<thumb-drive-name>/<current-image-name.img.gz> | dd bs=4M of=/dev/sda status=progress`
    - You can hit the tab key for autocompletion. This may help in finding your thumbdrive.
12. Wait for the command to complete
    - The dd command will take a long time to run.
    - The dd param `status=progress` should provide feedback on progress.
    - When the command is complete it will tell you how long it took
13. Power off the pc
14. Unplug all of the USBs
15. Power on the pc

## How to create a new VBXC PC image

### Fedora 26+ Base

1. Configure the OS exactly how you would like it to be on any system you plan to image
2. Boot fedora from a Live USB
    - Plug Live USB in
    - Hold the delete key while powering-up the NUC
    - Select your usb from the "boot override" section of the "save and exit" tab (Use arrow keys to navigate)
    - Hit enter
    - Select "Start fedora live" from the menu (Should be highlighted in white)
    - Wait for the system to boot
3. Click on the install fedora button and follow the instructions.
4. Reboot the nuc and unplug the live USB
5. Configure the OS. Here's a checklist: (Versabuilt's Fedora Doc's might be helpful)
    - Configure network adapters and settings
    - [Configure the firewall](/procedures/network-settings.html)
    - [Fedora image steps](/documentation/fedora-image.html)
    - Install VBXC base dependencies
    - Prevent the PC from from going to sleep, or turning off the display.
    - Set chromium to default browser
    - Change wallpaper to VersaBuilt
    - Add vbxc.desktop to /home/vbxc/.config/autostart
6. Plug in a USB with formatted with the linux filesystem
7. Open a terminal
8. Switch to the root user
9. type `dd bs=4M if=/dev/sda status=progress | gzip > /run/media/<USB>/<image-name>.img.gz`
10. Wait for it to finish
11. The new image is on your USB.
