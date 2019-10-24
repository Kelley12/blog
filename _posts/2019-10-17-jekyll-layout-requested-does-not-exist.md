---
title: "Jekyll Layout Requested Does Not Exist"
author: bkelley
layout: post
date: 2019-10-17 10:50:42 -0600
categories: posts
image: /assets/posts/jekyll-build-warning.png
description: >
  Jekyll "Build Warning Layout 'blog' requested in index.html does not exist"
---

- [The Problem](#the-problem)
- [Verify Build From Root Directory](#verify-build-from-root-directory)
- [Verbose Build](#verbose-build)
- [Check Theme _layouts folder](#check-theme-_layouts-folder)
- [Updating Jekyll](#updating-jekyll)
- [Validate Ruby Version](#validate-ruby-version)
- [Reinstall Jekyll](#reinstall-jekyll)
- [Thoroughly Inspect _config.yml](#thoroughly-inspect-_config.yml)

## The Problem

Seemingly out of nowhere, when trying to serve up my local jekyll instance it started throwing these build warnings:

```bash
Build Warning: Layout 'post' requested in _posts/2019-10-17-jekyll-layout-requested-does-not-exist.md does not exist.
Build Warning: Layout 'default' requested in 404.html does not exist.
Build Warning: Layout 'blog' requested in index.html does not exist.
```

While these may only be warnings, this breaks the site entirely. Because it cannot find the Layout, the pages are either just plain content or in the case of `index.html` a blank page.

***Note: This is only breaking on my local instance, GitHub pages still builds and renders everything without issue***

In order to try and figure out what was causing the error, I followed the following steps:

1. [Verify Build From Root Directory](#verify-build-from-root-directory)
2. [Run the build with verbosity](#verbose-build)
3. [Check theme _layouts folder](#check-theme-_layouts-folder)
4. [Updating Jekyll to the latest supported version](#updating-jekyll)
5. [Validate Ruby Version](#validate-ruby-version)
6. [Reinstall Jekyll](#reinstall-jekyll)
7. [Thoroughly Inspect _config.yml](#thoroughly-inspect-_config.yml)

## Verify Build From Root Directory

Some Jekyll users were experiencing this issue if building or serving Jekyll from a directory other than the root directory of the project (e.g. /posts). Verify that you are in the root directory. This was not my issue.

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

## Updating Jekyll

Just to verify that this wasn't a versioning issue or something like that, I updated to the latest version of jekyll supported by GitHub pages (found [here](https://pages.github.com/versions/)).

```bash
# Check the currently installed version of jekyll
bundle exec jekyll -v

# Update the gem to the latest supported version (3.8.5 in my case)
gem install jekyll -v 3.8.5
```

Unfortunately, this did not solve my problem.

## Validate Ruby Version

Made sure that the version of Ruby that was installed was up to date, supported by GitHub pages, and being used by RVM (Ruby Version Manager).

1. Check that the most recent, [GitHub Pages](https://pages.github.com/versions/) supported version of Ruby is installed. If not, use [these instructions to install the latest version](http://blog.allpunsindented.com/posts/2019/10/13/update-ruby-version.html).
2. Set the latest version as the default version

```bash
# Check currently installed Ruby version
ruby -v

# Make sure RVM is using the latest version
rvm --default use ruby-[version]

# Refresh rvm in the terminal
source ~/.rvm/scripts/rvm

# Install gems and try again
bundle install
bundle exec jekyll serve
```

Unfortunately for me I had the latest version installed and used by RVM.

## Reinstall Jekyll

Unlikely that this was the issue, but I had to ask myself "Have you tried turning it off and back on again". So I tried uninstalling and reinstalling Jekyll.

```bash
gem uninstall jekyll

# If you don't have permissions
sudo gem uninstall jekyll
```

If you have multiple versions of Jekyll installed, you will be prompted to select which version to uninstall.

```bash
Select gem to uninstall:
 1. jekyll-3.8.5
 2. jekyll-3.8.6
 3. jekyll-4.0.0
 4. All versions
>
```

Once all versions of jekyll were uninstalled, I attempted to reinstall it to the latest [GitHub Pages](https://pages.github.com/versions/) supported version.

```bash
gem install jekyll -v 3.8.5
```

As suspected, this did not fix my issue, but was worth a try.

## Thoroughly Inspect _config.yml

Running out of options for what could have gone wrong, I looked to things that I may have changed and didn't realize. While all of the gems seemed to be working right, I looked to the `_config.yml` file as it handles the theme and build settings.

For me, I am using a remote theme which requires a `remote_theme` configuration variable that specifies the theme repo:

```bash
remote_theme: hydecorp/hydejack
```

When I was committing code for the blog, I was getting these error messages from github:

```text
The page build completed successfully, but returned the following warning for the `master` branch:

You are attempting to use a Jekyll theme, "jekyll-theme-hydejack", which is not supported by GitHub Pages. Please visit https://pages.github.com/themes/ for a list of supported themes. If you are using the "theme" configuration variable for something other than a Jekyll theme, we recommend you rename this variable throughout your site. For more information, see https://help.github.com/en/articles/adding-a-theme-to-your-github-pages-site-using-jekyll.

For information on troubleshooting Jekyll see:

  https://help.github.com/articles/troubleshooting-jekyll-builds
```

In the link [Adding a theme to your GitHub pages site using Jekyll](https://help.github.com/en/articles/adding-a-theme-to-your-github-pages-site-using-jekyll) above, it has you replace the  `theme` configuration variable with the `remote_theme` configuration variable. Unfortunately the build in GitHub still worked and rendered without issue. I looked at adding this back in a couple times through debugging but kept adding in the repo name instead of the theme name itself. After adding back in the theme name, everything worked as expected, lesson learned the hard way.

```bash
theme: jekyll-theme-hydejack
```
