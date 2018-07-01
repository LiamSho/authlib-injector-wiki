<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
目录
=================

- [原版服务端、Spigot 等](#%E5%8E%9F%E7%89%88%E6%9C%8D%E5%8A%A1%E7%AB%AFspigot-%E7%AD%89)
- [BungeeCord](#bungeecord)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

本文主要介绍如何在 Minecraft 服务端使用 authlib-injector。

## 获取 authlib-injector
首先你需要从[此处](https://authlib-injector.yushi.moe/~download/)下载最新版本的 authlib-injector。

## 原版服务端、Spigot 等
设置 `online-mode` 为 `true`，然后[指定 `-javaagent` 参数](https://github.com/yushijinhun/authlib-injector/wiki/启动器技术规范#添加-jvm-参数)即可：
```
-javaagent:{authlib-injector.jar 的路径}={Yggdrasil 服务端的 URL（API Root）}
```

例如，这是原先的启动命令行：
```
java -jar minecraft_server.1.12.2.jar nogui
```
如果我要使用的 Yggdrasil 服务端的 API Root 为 `https://example.yggdrasil.yushi.moe/`，那么我需要将下载到的 authlib-injector 放在当前目录下（设其文件名为 `authlib-injector.jar`），然后添加 `-javaagent` 参数（必须置于 `-jar` 前）。

添加参数后的命令行如下：
```
java -javaagent:authlib-injector.jar=https://example.yggdrasil.yushi.moe/ -jar minecraft_server.1.12.2.jar nogui
```

## BungeeCord
如果使用 BungeeCord，那么在所有服务端上都需要加载 authlib-injector（[方法见上](#原版服务端spigot-等)），但应只有 BungeeCord 打开 `online-mode`，其它服务端应关闭 `online-mode`。
