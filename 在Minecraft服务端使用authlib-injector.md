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

**首先，你需要在 `server.properties` 中将 `online-mode` 设置为 `true`。**

接下来，你需要编辑 Minecraft 服务端的启动脚本（在 Windows 下其文件后缀名一般为 `.bat`，使用文本编辑器打开即可），启动脚本一般如下：

```
java -Xmx1024M -jar minecraft_server.1.12.2.jar nogui
```

你的启动脚本可能会与上例不同，但你只需**在 `-jar` 前**插入以下参数即可：

```
-javaagent:{path/to/authlib-injector.jar}={https://your-yggdrasil-api-root.com}
```

* `{path/to/authlib-injector.jar}` 表示你下载的 authlib-injector JAR 文件的路径（相对路径与绝对路径皆可）。
* `{https://your-yggdrasil-api-root.com}` 表示 Yggdrasil 服务端的 API Root。

例如：
* 你下载到的 authlib-injector JAR 文件名为 `authlib-injector-1.1.18-daa6fb4.jar`。
* 你将其放到了与服务端 JAR `minecraft_server.1.12.2.jar` 相同的目录下。
* Yggdrasil 服务端的 API Root 为 `https://example.com/api/yggdrasil`。

以上面的启动脚本为例，其修改后内容如下：
```
java -Xmx1024M -javaagent:authlib-injector-1.1.18-daa6fb4.jar=https://example.com/api/yggdrasil -jar minecraft_server.1.12.2.jar nogui
```

## BungeeCord

如果使用 BungeeCord，那么在所有服务端上都需要加载 authlib-injector（[方法见上](#原版服务端spigot-等)），但应只有 BungeeCord 打开 `online-mode`，其它服务端应关闭 `online-mode`。
