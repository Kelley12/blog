---
title: "Jekyll Theme SASS Error"
author: bkelley
layout: post
date: 2020-0- 16:04:42 -0600
categories: posts
image: /assets/posts/jekyll-theme.png
description: >
  Jekyll theme gem error "File to import not found or unreadable"
---

- [The Problem](#the-problem)

## The Problem

Just after I had created my first [jekyll theme](https://github.com/Kelley12/lucid-jekyll-theme) and published the gem to [rubygems.org](https://rubygems.org/gems/lucid-jekyll-theme) I tried to use it in my new jekyll site and ran into the following error

```bash
Conversion error: Jekyll::Converters::Scss encountered an error while converting 'assets/css/main.scss':
                File to import not found or unreadable: base. on line 10
jekyll 3.8.6 | Error:  File to import not found or unreadable: base. on line 10
```

This error only occurs when atempting to use the gem in the jekyll site and not while running the theme locally leaving no way to debug this theme

## The Setup

The setup in the new jekyll site is the same as it was done for this blog site and the other sites using remote themes.

### Gemfile

```ruby
gem 'lucid-jekyll-theme'
```

### _config.yml

```yml
theme: lucid-jekyll-theme
remote-theme: kelley12/lucid-jekyll-theme
```
