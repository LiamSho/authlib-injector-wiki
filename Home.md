<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
目录
=================

- [简介](#%E7%AE%80%E4%BB%8B)
- [相关项目](#%E7%9B%B8%E5%85%B3%E9%A1%B9%E7%9B%AE)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## 简介
该项目的目标：
 * 为修改 Minecraft ，使其使用自定义 Yggdrasil 服务提供工具
 * 为自定义 Yggdrasil 服务端、使用自定义 Yggdrasil 服务的启动器提供技术规范
 * 为玩家提供统一的非 Mojang 游戏外登录体验
   * 玩家可以使用任意实现该规范的启动器，登录任意实现该规范的 Yggdrasil 服务

该项目会对所有 API 作出详细说明，并且还会定义一些不属于 Yggdrasil 的 API 。这样做是为了最简化指定 Yggdrasil 服务的流程：只需要填写 Yggdrasil 服务对应的 URL ，就可以使用它。

## 相关项目
 * [yggdrasil-mock](https://github.com/yushijinhun/yggdrasil-mock)
   * Yggdrasil 服务端规范的参考实现，以及 Yggdrasil API 的测试用例
   * 基于此项目的 Yggdrasil 服务端演示站点：[example.yggdrasil.yushi.moe](https://github.com/yushijinhun/yggdrasil-mock/wiki/演示站点)
 * [BMCLAPI](https://bmclapidoc.bangbang93.com/#api-Mirrors-Mirrors_authlib_injector)
   * BMCLAPI 为 authlib-injector 下载提供了一个镜像
 * [Yggdrasil API for Blessing Skin](https://github.com/printempw/yggdrasil-api)
   * Blessing Skin 皮肤站的 Yggdrasil 插件
 * [HMCL](https://github.com/huanghongxun/HMCL)
   * HMCL v3 支持本规范
