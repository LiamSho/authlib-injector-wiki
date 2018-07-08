<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
目录
=================

- [获取 authlib-injector](#%E8%8E%B7%E5%8F%96-authlib-injector)
- [原版服务端、Spigot 等](#%E5%8E%9F%E7%89%88%E6%9C%8D%E5%8A%A1%E7%AB%AFspigot-%E7%AD%89)
- [BungeeCord](#bungeecord)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

本文主要介绍如何在 Minecraft 服务端使用 authlib-injector。

## 获取 authlib-injector

首先你需要从[此处](https://authlib-injector.yushi.moe/~download/)下载最新版本的 authlib-injector。

## 原版服务端、Spigot 等

请将服务端的 `online-mode` 设置为 `true`，然后在其启动命令中添加以下 JVM 参数：

```
-javaagent:{path/to/authlib-injector.jar}={https://your-yggdrasil-api-root.com}
```

- `{path/to/authlib-injector.jar}` 表示你在[上一步](获取-authlib-injector)中下载的 JAR 文件所在的位置（相对路径、绝对路径皆可）。
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
