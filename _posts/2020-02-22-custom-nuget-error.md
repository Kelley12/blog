---
title: "Custom NuGet Error Message"
author: bkelley
layout: post
date: 2020-02-22 08:08:00 -0600
categories: posts
image: /assets/posts/custom-error-message.png
description: >
  Cutom error when developer is missing company NuGet source
---

If you have a private NuGet repo or use a local network repo, when a new developer checks out the code a attempts to run it, they will get the generic NuGet error:

```txt
This project references NuGet package(s) that are missing on this computer. Use NuGet Package Restore to download them.  For more information, see http://go.microsoft.com/fwlink/?LinkID=322105. The missing file is {0}.
```

Even when the developer does a `NuGet Package Restore`, the source for the private or local repo will not be available.

You might document this package source in onboarding documentation or the like, but steps are skipped and overlooked. An alternative to this is to explicitely throw an error telling the user to check for the custom source.

In the project `.csproj` file, inside the `<Project>` tag, enter the following with whatever `ErrorText` value you want.

```xml
<Target Name="EnsureNuGetPackageBuildImports" BeforeTargets="PrepareForBuild">
    <PropertyGroup>
      <ErrorText>This project references NuGet package(s) that are missing on this computer. Use NuGet Package Restore to download them.  For more information, see http://go.microsoft.com/fwlink/?LinkID=322105. The missing file is {0}.</ErrorText>
    </PropertyGroup>
</Target>
```

Example custom message:

```xml
<Target Name="EnsureNuGetPackageBuildImports" BeforeTargets="PrepareForBuild">
    <PropertyGroup>
      <ErrorText>This project references NuGet package(s) missing on this computer. Use NuGet Package Restore to download them. Ensure that the \\myserver\packages source is in NuGet Package Sources. For more information, see http://go.microsoft.com/fwlink/?LinkID=322105. The missing file is {0}.</ErrorText>
    </PropertyGroup>
</Target>
```
