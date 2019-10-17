---
title: "Update Ruby Version"
layout: post
date: 2019-10-13 1:32:42 -0600
categories: posts
image: /assets/posts/ruby-version.png
description: >
  Update Ruby to latest of specified version
---

## Check that Ruby is installed

Open a terminal and type the following command. This should give you the latest version if Ruby is installed or a `command not found` error if it is not.

```bash
ruby -v
```

## Install the Ruby Version Manager

Install the Ruby version manager (rvm) to manage the version of Ruby that is used. Install or update to the latest stable version by executing the following command.

```bash
curl -L https://get.rvm.io | bash -s stable
```

If you are installing rvm for the first time, a terminal restart may be required. Or run the following command to refresh

```bash
source ~/.rvm/scripts/rvm
```

## Install the Desired Ruby Version

Check the [Ruby Website](https://www.ruby-lang.org/en/downloads/) for the latest stable version of Ruby. Or use the following command to install the latest version.

```bash
rvm install ruby --latest
```

If you want to install a different version of Ruby than the latest, you can use the following command to see the available versions.

*Ruby versions under the ***MRI Rubies*** section of the results*

```bash
rvm list known
```

Then use the desired version to run the following command

```bash
rvm install ruby-[version]
```

This may take some time to install depending on the number of dependencies and may ask for permission a few times.

## Set the Version of Ruby in RVM

Now that the desired version of Ruby is installed it should be the version being run. Verify by running the following command.

```bash
ruby -v
```

If the version returned is not the desired version, rvm needs to be set to know which version should be used. Use the following command to set the desired version as the default version.

```bash
rvm --default use ruby-[version]
```

You can use multiple versions of Ruby on different project by telling rvm which version of Ruby to use at any time by running the following command.

```bash
rvm use ruby-[version]
```
