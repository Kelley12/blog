---
title: "Jekyll Implicit Conversion Error"
author: bkelley
layout: post
date: 2019-10-24 1:44:42 -0600
categories: posts
image: /assets/posts/jekyll-implicit-conversion.png
description: >
  Jekyll Error "no implicit conversion of nil into String"
---

- [The Problem](#the-problem)
- [Jekyll Doctor](#jekyll-doctor)
- [Verbose Build](#verbose-build)
- [Stack Trace](#stack-trace)
- [Tracking Down the Error](#tracking-down-the-error)

## The Problem

While trying to create a new Jekyll website to host on GitHub pages I tried using a remote theme which I have done in the past (For this blog in fact). But when setting the `theme` and `remote_theme` configuration variables to the remote theme in `_config.yml` it resulted in the following error

```bash
Configuration file: /Users/bkelley/dev/site/_config.yml
jekyll 3.8.6 | Error:  no implicit conversion of nil into String
```

Leaving the `theme` configuration variable as the default `minima` theme, this error does not occur and everything builds and serves as expected.

## Jekyll Doctor

The first thing I tried was to run the `jekyll doctor` command to check the configuration file.

```bash
jekyll doctor
```

This resulted in the same error message

```bash
Configuration file: /Users/bkelley/dev/site/_config.yml
jekyll 3.8.6 | Error:  no implicit conversion of nil into String
```

If the command was successful, it give the following message

```bash
Configuration file: /Users/bkelley/dev/site/_config.yml
  Your test results are in. Everything looks fine.
```

## Verbose Build

```bash
bundle exec jekyll build --verbose
```

Resulted in the following result

```bash
Logging at level: debug
Configuration file: /Users/bkelley/dev/jessica/_config.yml
             Theme: freelancer-theme-jekyll
      Theme source: /Users/bkelley/.rvm/gems/ruby-2.6.3/gems/freelancer-theme-jekyll-1.0.0
         Requiring: jekyll-paginate
         Requiring: jekyll-sitemap
         Requiring: jekyll-gist
         Requiring: jekyll-feed
         Requiring: jekyll-data
         Requiring: jemoji
         Requiring: jekyll-feed
jekyll 3.8.6 | Error:  no implicit conversion of nil into String
```

## Stack Trace

```bash
jekyll build --trace
```

Resulted in the following result

```bash
Configuration file: /Users/bkelley/dev/jessica/_config.yml
Traceback (most recent call last):
        20: from /Users/bkelley/.rvm/gems/ruby-2.6.3/bin/ruby_executable_hooks:24:in `<main>`
        19: from /Users/bkelley/.rvm/gems/ruby-2.6.3/bin/ruby_executable_hooks:24:in `eval`
        18: from /Users/bkelley/.rvm/gems/ruby-2.6.3/bin/jekyll:23:in `<main>`
        17: from /Users/bkelley/.rvm/gems/ruby-2.6.3/bin/jekyll:23:in `load`
        16: from /Users/bkelley/.rvm/gems/ruby-2.6.3/gems/jekyll-3.8.6/exe/jekyll:15:in `<top (required)>`
        15: from /Users/bkelley/.rvm/gems/ruby-2.6.3/gems/mercenary-0.3.6/lib/mercenary.rb:19:in `program`
        14: from /Users/bkelley/.rvm/gems/ruby-2.6.3/gems/mercenary-0.3.6/lib/mercenary/program.rb:42:in `go`
        13: from /Users/bkelley/.rvm/gems/ruby-2.6.3/gems/mercenary-0.3.6/lib/mercenary/command.rb:220:in `execute`
        12: from /Users/bkelley/.rvm/gems/ruby-2.6.3/gems/mercenary-0.3.6/lib/mercenary/command.rb:220:in `each`
        11: from /Users/bkelley/.rvm/gems/ruby-2.6.3/gems/mercenary-0.3.6/lib/mercenary/command.rb:220:in `block in execute`
        10: from /Users/bkelley/.rvm/gems/ruby-2.6.3/gems/jekyll-3.8.6/lib/jekyll/commands/build.rb:18:in `block (2 levels) in init_with_program`
         9: from /Users/bkelley/.rvm/gems/ruby-2.6.3/gems/jekyll-3.8.6/lib/jekyll/commands/build.rb:30:in `process`
         8: from /Users/bkelley/.rvm/gems/ruby-2.6.3/gems/jekyll-3.8.6/lib/jekyll/commands/build.rb:30:in `new`
         7: from /Users/bkelley/.rvm/gems/ruby-2.6.3/gems/jekyll-3.8.6/lib/jekyll/site.rb:34:in `initialize`
         6: from /Users/bkelley/.rvm/gems/ruby-2.6.3/gems/jekyll-3.8.6/lib/jekyll/hooks.rb:102:in `trigger`
         5: from /Users/bkelley/.rvm/gems/ruby-2.6.3/gems/jekyll-3.8.6/lib/jekyll/hooks.rb:102:in `each`
         4: from /Users/bkelley/.rvm/gems/ruby-2.6.3/gems/jekyll-3.8.6/lib/jekyll/hooks.rb:103:in `block in trigger`
         3: from /Users/bkelley/.rvm/gems/ruby-2.6.3/gems/jekyll-data-1.1.0/lib/jekyll-data.rb:50:in `block in <top (required)>`
         2: from /Users/bkelley/.rvm/gems/ruby-2.6.3/gems/jekyll-data-1.1.0/lib/jekyll-data.rb:50:in `new`
         1: from /Users/bkelley/.rvm/gems/ruby-2.6.3/gems/jekyll-data-1.1.0/lib/jekyll-data/reader.rb:10:in `initialize`
/Users/bkelley/.rvm/gems/ruby-2.6.3/gems/jekyll-data-1.1.0/lib/jekyll-data/reader.rb:10:in `join`: no implicit conversion of nil into String (TypeError)
```

The last line in the stack trace gives the file, line number, and function the error occurred in. Using this, I could go see the line and figure out why this error was occurring.

## Tracking Down the Error

Using the stack trace I was able to get the file the error occurred in `reader.rb` in the `jekyll-data` plugin. Using VS Code, I was able to simply Cmd+click the line to follow it to the file, to the line that threw the error. You could do this by using the file explorer as well. In doing so, I was routed to this line:

```ruby
@theme_data_files = Dir[File.join(site.theme.data_path, "**", "*.{yaml,yml,json,csv,tsv}")]
```

Based on the error message, the error occurred in the `join` function. Also, the error message stated `o implicit conversion of nil into String` and seeing that 2 of the 3 parameters passed to join were in fact strings, `site.theme.data_path` must be the culprit.

In doing some research on this issue seeing if anyone else had run into this, I came across an [issue](https://github.com/ashmaroli/jekyll-data/issues/34) created in the `jekyll-data` plugin repository. There was an assumption that if you were using this plugin, your theme would have a `_data` directory so there wasn't a null check on `site.theme.data_path`. This was resolved in `v1.1.0`.

The Jekyll theme that I was using did not have a `_data` directory at the root. Unfortunately adding a data directory at the root of my project did not fix this either. So I created an [issue](https://github.com/jeromelachaud/freelancer-theme/issues/107) on the Jekyll theme to get a data directory added or get the dependency removed if unused.
