---
title: "Dynamic app.config Values"
author: bkelley
layout: post
date: 2020-03- 08:08:00 -0600
categories: posts
image: /assets/posts/.png
description: >
  Dynamicaly change app.config values based on configuration
---

### Dynamic app.config

If you've ever run into an instance where you want to use

Include a Debug config file and add the app.config transoformation

```xml
<Target Name="App_config_AfterCompile" AfterTargets="AfterCompile" Condition="Exists('$(ProjectName).exe.$(Configuration).config')">

    <!--Generate transformed app config in the intermediate directory-->
    <TransformXml Source="App.config" Destination="$(IntermediateOutputPath)$(TargetFileName).config" Transform="$(ProjectName).exe.$(Configuration).config" />

    <!--Force build process to use the transformed configuration file from now on.-->
    <ItemGroup>
        <AppConfigWithTargetPath Remove="App.config" />
        <AppConfigWithTargetPath Include="$(IntermediateOutputPath)$(TargetFileName).config">
            <TargetPath>$(TargetFileName).config</TargetPath>
        </AppConfigWithTargetPath>
    </ItemGroup>
</Target>

<!--Override After Publish to support ClickOnce AfterPublish. Target replaces the untransformed config file copied to the deployment directory with the transformed one.-->

<Target Name="App_config_AfterPublish" AfterTargets="AfterPublish" Condition="Exists('App.$(Configuration).config')">
    <PropertyGroup>
        <DeployedConfig>$(_DeploymentApplicationDir)$(TargetName)$(TargetExt).config$(_DeploymentFileMappingExtension)</DeployedConfig>
    </PropertyGroup>

    <!--Publish copies the untransformed App.config to deployment directory so overwrite it-->
    <Copy Condition="Exists('$(DeployedConfig)')" SourceFiles="$(IntermediateOutputPath)$(TargetFileName).config" DestinationFiles="$(DeployedConfig)" />
</Target>
```
