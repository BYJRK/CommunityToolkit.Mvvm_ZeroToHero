---
comments: true
---

# 在 .NET Framework 4.x 项目中使用源生成器

!!! warning
    虽然理论上是可以在 .NET Framework 项目中使用源生成器的，但是实际上并不推荐，因为需要对现有项目进行较大程度的改造，且稳定性并不能保证。因此，这里仅给出方案，但并不建议大家这样做。

本章节有对应的 B 站视频：

[:simple-bilibili: Bilibili](https://www.bilibili.com/video/BV188411g7ox/){ .md-button }

## 原理

[源生成器的文档](https://learn.microsoft.com/zh-cn/dotnet/csharp/roslyn-sdk/source-generators-overview) 指出，需要 .NET Standard 2.0 以上版本。同时可以发现，.NET Standard 2.0 支持的 [最低 .NET Framework 版本为 4.6.1](https://learn.microsoft.com/en-us/dotnet/standard/net-standard?tabs=net-standard-2-0)。.NET Standard 2.1 不支持 .NET Framework，所以无法使用更高的版本。

这时候我们不难产生一个想法：

1. 在一个 .NET Standard 2.0 项目中编写使用了源生成器的代码（也就是 ViewModel 为主的部分）
2. 在一个 .NET Framework 4.6.1 项目中引用这个 .NET Standard 2.0 项目

这样，我们就变相实现了在 .NET Framework 4.6.1 项目中使用源生成器了。

此外还有一个要求，我们的项目文件必须是 SDK 风格的项目文件，这样才能使用源生成器。对于这个问题，我们可以使用 [升级助手（Upgrade Assistant）](https://dotnet.microsoft.com/en-us/platform/upgrade-assistant)这个 CLI 工具来将现有的项目文件升级为 SDK 风格的项目文件。相关的方法大家可以查看上面的 B 站视频。

## 代价

虽然我们可以通过这种方式在 .NET Framework 4.6.1 项目中使用源生成器，但是这样做并不推荐。因为这样做会带来一些困难和未知的问题：

1. 项目结构变得复杂，需要将使用了源生成器的部分代码拆分到不同的项目中
2. 项目的稳定性无法保证，因为源生成器是一个新特性，可能会有一些未知的问题
3. 项目的维护成本增加，因为需要对现有项目进行一定程度的改造

因此，如果你的项目是一个新项目，或者可以接受将项目升级到 .NET 5.0+ 的话，建议直接使用 .NET 5.0+ 项目，而不是使用这种方式。

**但即便你使用了 .NET Framework，也并不意味着你与工具包无缘了**。工具包提供了非常多的功能，可以让你的开发变得更加便捷高效。并且可以配合 `PropertyChanged.Fody` 这样的库来实现属性通知（因为它会自动识别 `OnPropertyChanged` 这个名称的方法，从而为属性生成正确的 `setter` 的内容）。