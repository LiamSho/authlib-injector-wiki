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

**使用本外置登录方案，你必须在 `server.properties` 中将 `online-mode` 设置为 `true`。**

接下来，请编辑你的 Minecraft 服务端的启动脚本（使用 `.bat` 文件的用户请使用文本编辑器打开修改），启动服务端的脚本一般来说是长这样的：

```
java -Xmx1024M -Xms1024M -jar minecraft_server.1.12.2.jar nogui
```

你的启动命令可能与上面的示例有些不同，这是正常的。在你自己的启动命令中能找到类似 `-jar xxx.jar` 的部分即可。

authlib-injector 是通过 JVM 的 [`-javaagent`](https://github.com/yushijinhun/authlib-injector/wiki/%E5%90%AF%E5%8A%A8%E5%99%A8%E6%8A%80%E6%9C%AF%E8%A7%84%E8%8C%83#%E6%B7%BB%E5%8A%A0-jvm-%E5%8F%82%E6%95%B0) 参数加载的，因此你需要在你的启动命令的**正确的位置**上插入以下参数：

```
-javaagent:{path/to/authlib-injector.jar}={https://your-yggdrasil-api-root.com}
```

- `{path/to/authlib-injector.jar}` 表示你在上一步中下载的 `.jar` 文件所在的位置，可以是相对路径也可以是绝对路径。
- `{https://your-yggdrasil-api-root.com}` 表示的是你的 Yggdrasil 服务器的 API Root。

👇👇👇

**你必须把上述参数插入到 `-jar xxx.jar` 之前，否则 authlib-injector 不会被加载。**

👆👆👆

举个栗子，假设：

- 你下载到了文件名为 `authlib-injector-1.1.18-daa6fb4.jar` 的文件
- 并将其放到了与服务端核心 `minecraft_server.1.12.2.jar` 相同的目录下
- 你的 Yggdrasil 服务器 API Root 为 `https://example.com/api/yggdrasil`

那么你应该将上面的示例启动命令修改为这样：

```
java -Xmx1024M -Xms1024M -javaagent:authlib-injector-1.1.18-daa6fb4.jar=https://example.com/api/yggdrasil -jar minecraft_server.1.12.2.jar nogui
```

## BungeeCord

如果使用 BungeeCord，那么在所有服务端上都需要加载 authlib-injector（[方法见上](#原版服务端spigot-等)），但应只有 BungeeCord 打开 `online-mode`，其它服务端应关闭 `online-mode`。
