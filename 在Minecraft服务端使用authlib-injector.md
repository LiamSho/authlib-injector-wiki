<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
目录
=================

- [原版服务端、Spigot 等](#%E5%8E%9F%E7%89%88%E6%9C%8D%E5%8A%A1%E7%AB%AFspigot-%E7%AD%89)
- [BungeeCord](#bungeecord)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

本文主要介绍如何在 Minecraft 服务端使用 authlib-injector。

## 原版服务端、Spigot 等
设置 `online-mode` 为 `true`，然后[指定 javaagent](https://github.com/to2mbn/authlib-injector/wiki/启动器技术规范#添加-jvm-参数) 即可：
```
-javaagent:{authlib-injector.jar 的路径}={Yggdrasil 服务端的 URL（API Root）}
```

## BungeeCord
如果使用 BungeeCord，那么在所有服务端上都需要加载 authlib-injector（[方法见上](#原版服务端spigot-等)），但应只有 BungeeCord 打开 `online-mode`，其它服务端应关闭 `online-mode`。
