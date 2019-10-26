---
title: "Debugging Jekyll Sites"
author: bkelley
layout: post
date: 2019-10-25 23:04:42 -0600
categories: posts
image: /assets/posts/debugging-jekyll.png
description: >
  How to debug your Jekyll site when you run into unknown errors
---

- [Jekyll Doctor](#jekyll-doctor)
- [Verbose Build](#verbose-build)
- [Stack Trace](#stack-trace)
- [Gem Inspection](#gem-inspection)

## Jekyll Doctor

The Jekyll [doctor](https://jekyllrb.com/docs/usage/) command checks over the configuration file for any deprecation or configuration issues.

```bash
jekyll doctor
# or
jekyll hyde
```

## Verbose Build

Build your Jekyll site with the [verbose output option](https://jekyllrb.com/docs/configuration/options/#build-command-options) to see all build messages.

```bash
bundle exec jekyll build --verbose
```

## Stack Trace

Build your Jekyll site with the trace option which is not listed as an option on the [Jekyll build command options](https://jekyllrb.com/docs/configuration/options/#build-command-options). This will show the stack trace of any errors that occur when building your Jekyll site.

```bash
jekyll build --trace
```

## Gem Inspection

Sometimes you have to verify that a particular gem contains a file or folder expected by Jekyll or a Jekyll plugin. The following command allows you to peek at the gem file structure

```bash
ls $(bundle show [gem])
```
