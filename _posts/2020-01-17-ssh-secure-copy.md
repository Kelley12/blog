---
title: "SSH Secure copy"
author: bkelley
layout: post
date: 2020-01-17 06:24:00 -0600
categories: posts
image: /assets/posts/ssh-secure-copy.png
description: >
  Securely copy files over SSH
---

## Prerequisites

On the machine that you want to make a copy from, open the terminal. Then run one of the commands below.

## Copy a single file

```bash
scp /path/to/file username@machine.ip:/path/to/destination

# example
scp config.json root@123.45.67.8:~/.configs
```

## Copy multiple files at once

```bash
scp /path/to/file1 /path/to/file2 username@machine.ip:/path/to/destination

# example
scp config1.json config2.json root@123.45.67.8:~/.configs
```
