<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
目录
=================

- [简介](#%E7%AE%80%E4%BB%8B)
- [本项目与authlib-agent的关系](#%E6%9C%AC%E9%A1%B9%E7%9B%AE%E4%B8%8Eauthlib-agent%E7%9A%84%E5%85%B3%E7%B3%BB)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## 简介
该项目的目标：
 * 为修改Minecraft，使其使用**自定义**Yggdrasil服务提供工具
 * 为自定义Yggdrasil服务端、使用自定义Yggdrasil服务的启动器提供**技术规范**
 * 为玩家提供**统一的**非Mojang游戏外登录体验
   * 玩家可以使用**任意**实现该规范的启动器，登录**任意**实现该规范的Yggdrasil服务

该项目会对所有API作出详细说明，并且还会定义一些**不属于**Yggdrasil的API。这样做是为了**最简化**指定Yggdrasil服务的流程：只需要填写Yggdrasil服务对应的URL，就可以使用它。

## 本项目与authlib-agent的关系
authlib-agent项目存在较多历史遗留问题，并且原项目的javaagent部分及后端部分耦合在一起，需要一起构建。因此将原项目的javaagent部分重写，并更名authlib-injector，同时提供更加友好的配置方式，以供其它yggdrasil服务端实现使用。
