---
title: "Error while creating mount source path"
author: bkelley
layout: post
date: 2023-10-27 19:39:00 -0600
categories: posts
image: /assets/posts/mount-source-path.png
description: >
  Dynamicaly change app.config values based on configuration
---

## Error

When running the docker container using docker compose, you may get the following error

```text
Error: (HTTP code 500) server error - error while creating mount source path '/host_mnt/Users/user/.docker/run/docker.sock': mkdir /host_mnt/Users/user/.docker/run/docker.sock: operation not supported
```

## Solution

```bash
sudo ln -sf "$HOME/.docker/run/docker.sock" /var/run/docker.sock
```

## Explanation

Tracking this one down took a little bit. The issue is that the docker.sock file is not accessible to the docker container. In order to find the problem, I ended renaming the docker.sock file and running the docker container. This gave me the following error

```text
Error: No Docker client strategy found
```

At this point I was able to have a better error to search with. This led me to the following [stackoverflow post](https://stackoverflow.com/questions/73976244) which gave me the solution.

## References

<https://stackoverflow.com/questions/73976244>
