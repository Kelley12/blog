---
title: "Dynamic app.config Values"
author: bkelley
layout: post
date: 2020-03-08 08:08:00 -0600
categories: posts
image: /assets/posts/dynamic-app-config.png
description: >
  Dynamicaly change app.config values based on configuration
---

## Dynamic app.config

Dynamically populating the values of the `app.config` can be extremely useful for usage in multiple environments with the quick change of a project configuration.

This can be accomplished with few simple steps.

1. [Modify App.config](#modify-app.config)
2. [Create Configuration Files](#create-configuration-files)
3. [Using Task and Targets](#using-task-and-targets)
4. [Item Group](#item-group)

Include a Debug config file and add the `app.config` transformation

### Modify App.config

Modify the `app.config` to set all of your variable values to be empty, `<value />`. These values will be changed upon build and clearing the values prevents any confusion.

The app config `ApplicationSettings` property should resemble the following

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  ...
  <applicationSettings>
    <ProjectName.Properties.Settings>
      <setting name="Variable1" serializeAs="String">
        <value />
      </setting>
      <setting name="Variable2" serializeAs="String">
        <value />
      </setting>
    </ProjectName.Properties.Settings>
  </applicationSettings>
</configuration>
```

### Create Configuration Files

In this tutorial, I am using the configuration naming convention of `ProjectName.Configuration.config`. For example, if my project was called `AdventureWorks`, the `debug` configuration would be `AdventureWorks.debug.config`.

Create configuration files using the naming convention above for each of the configurations you have.

***Note: If you choose to got with a different naming convention, you will need to make more changes in the following steps that I will outline in each step.***

The confiuration files should look like the following

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <applicationSettings>
    <ProjectName.Properties.Settings>
      <setting name="Variable1" serializeAs="String">
        <value>Variable1 Value</value>
      </setting>
      <setting name="Variable2" serializeAs="String">
        <value>Variable2 Value</value>
      </setting>
    </ProjectName.Properties.Settings>
  </applicationSettings>
</configuration>
```

### Using Task and Targets

In the `.csproj` file, somehwere within the `<Project>` tag, add the following code:

```xml
<UsingTask TaskName="TransformXml" AssemblyFiles="$(MSBuildExtensionsPath32)\Microsoft\VisualStudio\v$(VisualStudioVersion)\Web\Microsoft.Web.Publishing.Tasks.dll" />
<Target Name="App_config_AfterCompile" AfterTargets="AfterCompile" Condition="Exists('$(ProjectName).$(Configuration).config')">

    <!--Generate transformed app config in the intermediate directory-->
    <TransformXml Source="App.config" Destination="$(IntermediateOutputPath)$(TargetFileName).config" Transform="$(ProjectName).$(Configuration).config" />

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

***Note: If you are using a different naming convention, you will have to make the following changes: modify the `Condition` value of the `App_config_AfterCompile` target and `Transform` value of the `TransformXml` property to match your config naming convention.***

### Item Group

Inside the `.csproj` file, modify the `<ItemGroup>` section containing the `<None Include="App.config" />` property. Add each of the configurations so that is looks like the following:

```xml
<ItemGroup>
    <None Include="App.config" />
    <None Include="ProjectName.Debug.config">
      <DependentUpon>App.config</DependentUpon>
      <SubType>Designer</SubType>
    </None>

    <None Include="ProjectName.Release.config">
      <CopyToOutputDirectory>Always</CopyToOutputDirectory>
      <DependentUpon>App.config</DependentUpon>
      <SubType>Designer</SubType>
    </None>
    ...
</ItemGroup>
```

The important property for the configurations is the `DependentUpon` property which groups the config files within `app.config` in the solution explorer.

***Note: If you are using a different naming convention, you will have to change the `Include` value to match the name of each configuration.***
