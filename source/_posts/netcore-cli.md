---
title: .NET Core CLI命令
date: '2020-04-05 14:41:21'
updated: '2020-04-05 14:41:21'
categories:
  - C#
  - .NET Core
tags:
  - CLI
---
### Console

```shell
dotnet [command] [arguments] [--additional-deps] [--additionalprobingpath] [--depsfile]
    [-d|--diagnostics] [--fx-version] [-h|--help] [--info] [--roll-forward-on-no-candidate-fx]
    [--runtimeconfig] [-v|--verbosity] [--version]
```
<!--more -->
查看可用模版：
``` shell
dotnet new -l
```

例如创建一个MVC模版的项目：
``` shell
dotnet new mvc -lang c# -o my_web
```

### dotnet命令

#### 常规 

| 命令 | 说明 |
| --- | --- |
| dotnet build | 生成 .NET Core 应用程序。|
| dotnet clean | 清除生成输出。|
| dotnet help | 显示命令更详细的在线文档。|
| dotnet migrate | 将有效的预览版 2 项目迁移到 .NET Core SDK 1.0 项目。|
| dotnet msbuild | 提供对 MSBuild 命令行的访问权限。|
| dotnet new | 为给定的模板初始化 C# 或 F# 项目 |
| dotnet pack | 创建代码的 NuGet 包。|
| dotnet publish | 创建代码的 NuGet 包。 |
| dotnet restore | 还原给定应用程序的依赖项。|
| dotnet run | 从源运行应用程序。|
| dotnet sln | 用于添加、删除和列出解决方案文件中项目的选项。 |
| dotnet store | 将程序集存储到运行时包存储区。 | 
| dotnet test | 将程序集存储到运行时包存储区。 |

#### 项目引用

| 命令 | 说明 |
| --- | --- |
| dotnet add reference | 添加项目引用 |
| dotnet list reference | 列出项目引用 |
| dotnet remove reference | 删除项目引用

#### NuGet 包

| 命令 | 说明 |
| --- | --- |
| dotnet add package | 添加NuGet包 |
| dotnet remove package | 删除NuGet包 |

#### NuGet 命令

| 命令 | 说明 |
| --- | --- |
| dotnet nuget delete | 从服务器删除或取消列出包 |
| dotnet nuget locals | 清除或列出本地 NuGet 资源 |
| dotnet nuget push | 将包推送到服务器，并将其发布 |

#### 全局工具命令

| 命令 | 说明 |
| --- | --- |
| dotnet tool install | 在计算机上安装全局工具 | 
| dotnet tool list | 列出当前安装在计算机上的默认目录中或指定路径中的所有全局工具。|
| dotnet tool uninstall | 从计算机中卸载全局工具 |
| dotnet tool update  | 在计算机上更新全局工具 |
