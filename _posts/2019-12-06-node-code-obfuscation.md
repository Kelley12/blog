---
title: "Obfuscating NodeJS Code"
author: bkelley
layout: post
date: 2019-12-06 2:18:00 -0600
categories: posts
image: /assets/posts/node-code-obfuscation.png
description: >
  Obfuscating node.js code as a binary
---

1. Install [pkg](https://www.npmjs.com/package/pkg) (this guide assumes version 4.3.8):

   ```bash
   npm i -g pkg
   ```

2. Create a pkg.json config file. This will point pkg to any assets that need
   to be bundled with the code. For example, if there is a html frontend whose
   static files are hosted and read from a client folder in the top level
   of its repo, the resulting config would look like this:

   ```bash
   {
       "assets": [ "dist/client/**/*", "package.json" ]
   }
   ```

3. Compile the code. Point it to the root of the node code and the config
   file we just created. E.G.:

   ```bash
   pkg ./dist/server/index.js -c pkg.json
   ```

   or into package.json like:

   ```bash
   "compile": "npm run preversion && pkg ./dist/server/index.js -c pkg.json --targets linux-armv7 --output pkg/robot2cnc"
   ```

   *** NOTE: Crosscompilation between processors is not possible. Therefore you cannot compile on an x64 architecture (Mac or Windows) and run on armv7 (Raspberry Pi). A suggested solution is [here](https://github.com/zeit/pkg/issues/145). ***

4. This will create compiled, binary versions of the project.

    If `--targets` is not specified, it will create one for each OS: linux, Windows, and OSX

    - `index-linux`
    - `index-macos`
    - `index-win.exe`

    Find out more about targets in the [documentation](https://github.com/zeit/pkg#targets)

5. To use with Nitro or Robot2CNC, the Service will need to be edited to run the pkg

    To edit a service

    ```bash
    sudo systemctl edit --full vbxc.service

    OR

    sudo systemctl edit --full robot2cnc.service
    ```

    Then change the `ExecStart` setting to one of the following

    ```bash
    ExecStart=/home/vbxc/nitro/index-linux

    OR

    ExecStart=/home/pi/millipede/robot2cnc
    ```
