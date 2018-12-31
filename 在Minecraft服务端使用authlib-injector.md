<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
目录
=================

- [获取 authlib-injector](#%E8%8E%B7%E5%8F%96-authlib-injector)
- [原版服务端、Spigot 等](#%E5%8E%9F%E7%89%88%E6%9C%8D%E5%8A%A1%E7%AB%AFspigot-%E7%AD%89)
- [BungeeCord](#bungeecord)
- [调用 Mojang 皮肤](#%E8%B0%83%E7%94%A8-mojang-%E7%9A%AE%E8%82%A4)
- [插件 / Mod 兼容性](#%E6%8F%92%E4%BB%B6--mod-%E5%85%BC%E5%AE%B9%E6%80%A7)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

本文主要介绍如何在 Minecraft 服务端使用 authlib-injector。

## 获取 authlib-injector

首先你需要从[此处](https://authlib-injector.yushi.moe/~download/)下载最新版本的 authlib-injector。

## 原版服务端、Spigot 等

请将服务端的 `online-mode` 设置为 `true`，然后在其启动命令中添加以下 JVM 参数：

```
-javaagent:{path/to/authlib-injector.jar}={https://your-yggdrasil-api-root.com}
```

- `{path/to/authlib-injector.jar}` 表示你在[上一步](#获取-authlib-injector)中下载的 JAR 文件所在的位置（相对路径、绝对路径皆可）。
- `{https://your-yggdrasil-api-root.com}` 表示你的 Yggdrasil 服务端的 API Root。

例如，这是原先的启动命令：

```
java -jar minecraft_server.1.12.2.jar nogui
```

假设：

- 你下载到的 authlib-injector JAR 文件名为 `authlib-injector.jar`。
- 你将其放到了与服务端 JAR `minecraft_server.1.12.2.jar` 相同的目录下。
- 你的 Yggdrasil 服务端 API Root 为 `https://example.yggdrasil.yushi.moe`。

那么添加参数后的命令行应该如下：

```
java -javaagent:authlib-injector.jar=https://example.yggdrasil.yushi.moe -jar minecraft_server.1.12.2.jar nogui
```

## BungeeCord

如果使用 BungeeCord，那么在所有服务端上都需要加载 authlib-injector（[方法见上](#原版服务端spigot-等)），但应只有 BungeeCord 打开 `online-mode`，其它服务端应关闭 `online-mode`。

## 调用 Mojang 皮肤
> 此功能要求服务端和客户端的 authlib-injector 版本均满足 >=1.1.24（尚未发布）。

加载 authlib-injector 后，所有皮肤默认都是从指定的验证服务器处获取的。例如：
* `/give @p minecraft:skull 1 3 {SkullOwner:"notch"}`
* （Citizens2 插件）`/npc skin notch`

这些命令获取的都是**验证服务器上**名为 `notch` 的角色的皮肤。

如果要使用 Mojang 的皮肤，则可以在角色名称后加上 `@mojang`，如：
* `/give @p minecraft:skull 1 3 {SkullOwner:"notch@mojang"}`
* `/npc skin notch@mojang`

## 插件 / Mod 兼容性
一般而言，authlib-injector 兼容绝大多数插件和 Mod。下表列出的是（曾经）存在兼容性问题的插件 / Mod：

|受影响的插件 / Mod|受影响的 authlib-injector 版本|备注|
|----|---|----|
|Citizens2 (>=[build 1400](http://ci.citizensnpcs.co/job/Citizens2/1400/))|<=1.1.23|[#27](https://github.com/yushijinhun/authlib-injector/issues/27) [#28](https://github.com/yushijinhun/authlib-injector/pull/28)|

