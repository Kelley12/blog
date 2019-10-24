---
title: "Raspberry Pi Imaging"
author: bkelley
layout: post
date: 2019-10-22 10:56:42 -0600
categories: posts
image: /assets/posts/raspberry-pi-imaging.png
description: >
  Creating and writing a Raspberry Pi image to and from an SD card
---

- [Creating an image from an SD card](#creating-an-image-from-an-sd-card)
  - [Mac and Linux command line](#mac-and-inux-command-line)
  - [Windows - Win32 Disk Imager](#windows-win32-disk-imager)
- [Writing an image to an SD card](#writing-an-image-to-an-sd-card)
  - [Balena Etcher - Mac, Linux, and Windows](#balena-etcher-mac,-linux,-and-windows)
  - [Command line - Mac and Linux](#command-line-mac-and-linux)
  - [Win32 Disk Imager - Windows](#win32-disk-imager-windows)

## Creating an image from an SD card

### Mac and Linux command line

1. Use the following `diskutil` command to display the available disks on the system. This should display a list of disks, labeled as `/dev/diskn`.
    ***Note: The following examples use `/dev/disk2`.***

    ```bash
    diskutil list
    ```

2. Use the `dd` command and the `gzip` command to create a zip file of the image.

    **Mac**

    ```bash
    sudo dd if=/dev/disk2 | gzip > ~/path/to/disk/image.img.gz
    ```

    ***Note: Mac does not have a status parameter like linux but you can check the progress with `Ctrl+T`***

    **Linux**

    ```bash
    sudo dd bs=4M if=/dev/disk2 status=progress | gzip > ~/path/to/disk/image.img.gz
    ```

3. Once the image has finished writting, eject the disk

    ```bash
    sudo diskutil eject /dev/disk2
    ```

### Windows - Win32 Disk Imager

[Win32 Disk Imager](https://sourceforge.net/projects/win32diskimager/) is the preferred tool for writing an image to an SD card.

1. Choose a name and location for the image using the folder icon for the `Image File` field.
2. Select the device letter of the SD card in the `device` field the filename
3. Press `Read`

## Writing an image to an SD card

### Balena Etcher - Mac, Linux, and Windows

[Balena Etcher](https://www.balena.io/etcher/) is a simple SD card flasher for writing images.

1. Click `Select Image` and select the disk image file to write to the SD card
2. Click `Select Drive` if available or `Change` if the wrong SD card is selected and select the correct one.
3. Click `Flash` to start the image writing process

    ***Etcher defaults to automatically unmounting the drive or SD card after the image been successfully burned and written to the target volume, so keep that in mind if you go looking around in the Finder or elsewhere for a mounted image, it won’t be there. And yes, you can turn that off in Etcher app settings if need be.***

### Command line - Mac and Linux

1. Use the following `diskutil` command to display the available disks on the system. This should display a list of disks, labeled as `/dev/diskn`.
    ***Note: The following examples use `/dev/disk2`.***

    ```bash
    diskutil list
    ```

2. In order to write to the disk, it needs to first be unmounted.

    ```bash
    sudo diskutil unmountDisk /dev/disk2
    ```

3. Unzip the image using `gzip` and write it to the disk using the `dd` command as demonstrated in the following command.

    **Mac**

    ```bash
    gzip -dc ~/path/to/image/image.img.gz | sudo dd of=/dev/disk2
    ```

    ***Note: Mac does not have a status parameter like linux but you can check the progress with `Ctrl+T`***

    **Linux**

    ```bash
    gzip -dc ~/path/to/image/image.img.gz | sudo dd bs=4M of=/dev/disk2 status=progress
    ```

4. Once the image has finished writting, eject the disk

    ```bash
    sudo diskutil eject /dev/disk2
    ```

### Win32 Disk Imager - Windows

[Win32 Disk Imager](https://sourceforge.net/projects/win32diskimager/) is another tool for writing an image to an SD card.

1. Choose the image location using the folder icon for the `Image File` field.
2. Select the device letter of the SD card in the `device` field the filename
3. Press `Write`
