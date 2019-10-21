---
title: "Jekyll Layout Requested Does Not Exist"
layout: post
date: 2019-10-17 10:50:42 -0600
categories: posts
image: /assets/posts/jekyll-build-warning.png
description: >
  Jekyll "Build Warning Layout 'blog' requested in index.html does not exist"
---

- [The Problem](#the-problem)
- [Verbose Build](#verbose-build)
- [Check Theme _layouts folder](#check-theme-_layouts-folder)

## The Problem

Seemingly out of nowhere, when trying to serve up my local jekyll instance it started throwing these build warnings:

```bash
Build Warning: Layout 'post' requested in _posts/2019-10-17-jekyll-layout-requested-does-not-exist.md does not exist.
Build Warning: Layout 'default' requested in 404.html does not exist.
Build Warning: Layout 'blog' requested in index.html does not exist.
```

While this may only a warning, this breaks the site entirely. Because it cannot find the Layout, the pages are either just plain content or in the case of `index.html`, a blank page.

In order to try and figure out what was causing the error, I followed the following steps:

1. [Run the build with verbosity](#verbose-build)
2. [Check theme _layouts folder](#check-theme-_layouts-folder)

## Verbose Build

The first check was to see if there was any more information when building with more logging. To do so, I ran the build with the `--verbose` option using the following command:

```bash
bundle exec jekyll build --verbose
```

The output of this command resulted in this:

```bash
$ bundle exec jekyll build --verbose
  Logging at level: debug
Configuration file: /Users/bkelley/dev/blog/_config.yml
         Requiring: jekyll-feed
         Requiring: jekyll-paginate
            Source: /Users/bkelley/dev/blog
       Destination: /Users/bkelley/dev/blog/_site
 Incremental build: disabled. Enable with --incremental
      Generating...
       EntryFilter: excluded /Gemfile
       EntryFilter: excluded /Gemfile.lock
           Reading: _posts/2019-10-17-jekyll-layout-requested-does-not-exist.md
           Reading: _posts/2019-09-28-raspberry-pi-access-point.md
           Reading: _posts/2019-10-13-update-ruby-version.md
       Jekyll Feed: Generating feed for posts
        Generating: JekyllFeed::Generator finished in 0.00163 seconds.
        Generating: Jekyll::Paginate::Pagination finished in 0.000774 seconds.
  ...
         Rendering: _posts/2019-10-17-jekyll-layout-requested-does-not-exist.md
  Pre-Render Hooks: _posts/2019-10-17-jekyll-layout-requested-does-not-exist.md
  Rendering Markup: _posts/2019-10-17-jekyll-layout-requested-does-not-exist.md
  Rendering Layout: _posts/2019-10-17-jekyll-layout-requested-does-not-exist.md
     Build Warning: Layout 'post' requested in _posts/2019-10-17-jekyll-layout-requested-does-not-exist.md does not exist.
         Rendering: 404.html
  Pre-Render Hooks: 404.html
  Rendering Markup: 404.html
  Rendering Layout: 404.html
     Build Warning: Layout 'default' requested in 404.html does not exist.
         Rendering: index.html
  Pre-Render Hooks: index.html
  Rendering Markup: index.html
  Rendering Layout: index.html
     Build Warning: Layout 'blog' requested in index.html does not exist.
  ...
  Rendering Layout: feed.xml
           Writing: /Users/bkelley/dev/blog/_site/404.html
           Writing: /Users/bkelley/dev/blog/_site/index.html
           ...
           Writing: /Users/bkelley/dev/blog/_site/posts/2019/10/17/jekyll-layout-requested-does-not-exist.html
                    done in 0.496 seconds.
```

This unfortunately didn't help point out any issues.

## Check Theme _layouts folder

The next step was to check that the theme actually contains the requested layouts. In order to do so run the following command and replacing `[theme-gem]` with the theme gem you are using in `Gemfile`. If you're using minima, replace it with `minima`.

```bash
ls $(bundle show [theme-gem])/_layouts/
```

The output of this command with my theme `jekyll-theme-hydejack`:

```bash
$ ls $(bundle show jekyll-theme-hydejack)/_layouts/
about.html      blog.html       default.html    list.html       page.html       redirect.html
base.html       compress.html   home.html       not-found.html  post.html
```

As we can see, the build warnings for the missing themes `default`, `blog`, and `post` do exist in the themes _layouts folder.
