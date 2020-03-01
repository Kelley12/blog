---
title: "Visual Studio Project Macro Values"
author: bkelley
layout: post
date: 2020-03-01 08:55:52 -0600
categories: posts
image: /assets/posts/viewing-macro-variables.png
description: >
  Two methods for viewing the values of project macros
---

## Viewing Current Marcro Values

*The values shown are current values and do not reflect what they will be at build time.*

1. In Solution Explorer, right click on the project and select **Properties**

2. Select **Build Events** from the menu on the left

3. Click **Edit Pre-build...** or **Edit Post-build...**

4. Click on the **Macros >>** button

5. You should see a dialog with Macros and Values

## Printing Macros On Build

1. In Solution Explorer, right click on the project and select **Properties**

2. Select **Build Events** from the menu on the left

3. Enter `echo $(MacroName)` into the **Pre-build event command line:** or **Post-build event command line:** boxes depenending on when you want the macro value displayed

4. Build the project and view the output window
