---
title: "SSH Secure copy"
author: bkelley
layout: post
date: 2019-- ::00 -0600
categories: posts
image: /assets/posts/.png
description: >
  Securely copy files over SSH
---

## Prerequisites

On the machine that you want to make a copy from, open the terminal. Or simply remote into the machine if doing so remotely. Then run one of the commands below.

## Copy a single file

```bash
scp /path/to/file vbxc@172.86.160.00:/path/to/destination

example
scp parts.json vbxc@172.86.160.2:~/.config/nitro
```

## Copy multiple files at once

```bash
scp /path/to/file1 /path/to/file2 vbxc@172.86.160.00:/path/to/destination

example
scp parts.json processes.json vbxc@172.86.160.2:~/.config/nitro
```
