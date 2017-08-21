本文主要介绍如何在Minecraft服务端使用authlib-injector。

## 原版服务端、Spigot等
设置`online-mode`为`true`，然后[指定javaagent](https://github.com/to2mbn/authlib-injector/wiki/%E5%90%AF%E5%8A%A8%E5%99%A8%E6%8A%80%E6%9C%AF%E8%A7%84%E8%8C%83#%E6%B7%BB%E5%8A%A0jvm%E5%8F%82%E6%95%B0)即可：
```
-javaagent:{authlib-injector.jar的路径}=@{Yggdrasil服务端的URL（API Root）}
```

## BungeeCord
如果使用BungeeCord，那么在所有服务端上都需要加载authlib-injector（[方法见上](#原版服务端spigot等)），但应只有BungeeCord打开`online-mode`，其它服务端应关闭`online-mode`。