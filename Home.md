<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
目录
=================

- [简介](#%E7%AE%80%E4%BB%8B)
- [相关项目](#%E7%9B%B8%E5%85%B3%E9%A1%B9%E7%9B%AE)
- [本项目与 authlib-agent 的关系](#%E6%9C%AC%E9%A1%B9%E7%9B%AE%E4%B8%8E-authlib-agent-%E7%9A%84%E5%85%B3%E7%B3%BB)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## 简介
该项目的目标：
 * 为修改 Minecraft ，使其使用自定义 Yggdrasil 服务提供工具
 * 为自定义 Yggdrasil 服务端、使用自定义 Yggdrasil 服务的启动器提供技术规范
 * 为玩家提供统一的非 Mojang 游戏外登录体验
   * 玩家可以使用任意实现该规范的启动器，登录任意实现该规范的 Yggdrasil 服务

该项目会对所有 API 作出详细说明，并且还会定义一些不属于 Yggdrasil 的 API 。这样做是为了最简化指定 Yggdrasil 服务的流程：只需要填写 Yggdrasil 服务对应的 URL ，就可以使用它。

## 相关项目
 * [yggdrasil-mock](https://github.com/to2mbn/yggdrasil-mock)
    Yggdrasil 服务端规范的参考实现，以及 Yggdrasil API 的测试用例
	* 基于此项目的 Yggdrasil 服务端演示站点：[example.yggdrasil.yushi.moe](https://github.com/to2mbn/yggdrasil-mock/wiki/演示站点)
 * [Yggdrasil API for Blessing Skin](https://github.com/printempw/yggdrasil-api)
    Blessing Skin 皮肤站的 Yggdrasil 插件
 * [HMCL](https://github.com/huanghongxun/HMCL)
    HMCL v3 支持本规范

## 本项目与 authlib-agent 的关系
authlib-agent 项目存在较多历史遗留问题，并且原项目的 javaagent 部分及后端部分耦合在一起，需要一起构建。因此将原项目的 javaagent 部分重写，并更名 authlib-injector ，同时提供更加友好的配置方式，以供其它 Yggdrasil 服务端实现使用。
