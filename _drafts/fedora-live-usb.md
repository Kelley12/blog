---
title: "Fedora live USB"
author: bkelley
layout: post
date: 2020-0- ::00 -0600
categories: posts
image: /assets/posts/fedora-live-usb.png
description: >
  Creating a Fedora live USB image
---

## Creating a Fedora live USB image on Mac

1. [Download Fedora 64-bit live image](https://getfedora.org/en/workstation/download/)
2. Attach USB drive to computer
3. In the terminal type `diskutil list` and find your USB drive in the list (example /dev/disk3)
4. Unmount the drive with `sudo umount /dev/diskN`
5. Convert the Fedora iso to an img using hdiutil `hdiutil convert -format UDRW -o <path-to>/fedora2X.img <path-to>/fedora2X.iso`
6. If using OS X, it tends to put the .dmg ending on the output file automatically, rename the file using `mv <path-to>/fedora2X.img.dmg <path-to>/fedora2X.img`
7. Now dd the img file to the USB stick `sudo dd if=<path-to>/fedora2X.img of=/dev/diskN bs=1m status=progress`
8. Eject the USB drive from the computer properly `diskutil eject /dev/diskN`
