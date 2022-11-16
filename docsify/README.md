# 北京惺惺科技有限公司

<p align="center">
<img align="center" width="150px" src="https://raw.githubusercontent.com/cherish-chat/xx-doc/master/docsify/images/xxim@3x.webp">
</p>

北京惺惺科技有限公司刚刚成立，专门从事互联网产品的研发，目前主要是做IM相关的产品，后续会有更多的产品上线。
如果有定制化的需求，可以联系我们，我们会根据您的需求提供定制化的解决方案。

  > 技术服务、技术开发、技术咨询、技术交流、技术转让、技术推广；软件开发；人工智能基础软件开发；人工智能应用软件开发；计算机系统服务；数据处理服务；广告发布；广告设计、代理；广告制作；市场调查（不含涉外调查）；企业形象策划；社会经济咨询服务；会议及展览服务；组织文化艺术交流活动；计算机软硬件及辅助设备零售。（除依法须经批准的项目外，凭营业执照依法自主开展经营活动）许可项目：互联网信息服务；网络文化经营。（依法须经批准的项目，经相关部门批准后方可开展经营活动，具体经营项目以相关部门批准文件或许可证件为准）（不得从事国家和本市产业政策禁止和限制类项目的经营活动。）

# XXIM项目
<div align=center>

[![Go Report Card](https://goreportcard.com/badge/github.com/cherish-chat/xxim-server)](https://goreportcard.com/report/github.com/cherish-chat/xxim-server)
[![Release](https://img.shields.io/github/v/release/cherish-chat/xxim-server.svg?style=flat-square)](https://github.com/cherish-chat/xxim-server)
[![Go Reference](https://pkg.go.dev/badge/github.com/cherish-chat/xxim-server.svg)](https://pkg.go.dev/github.com/cherish-chat/xxim-server)
[![Awesome Go](https://cdn.rawgit.com/sindresorhus/awesome/d7305f38d29fed78fa85652e3a63e154dd8e8829/media/badge.svg)](https://github.com/avelino/awesome-go)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

</div>


## 🤷‍ xxim 介绍
简体中文 | [English](README-EN.md)

xxim-server代码不复杂，im大多逻辑都在于客户端，所以xxim-server只是一个简单的im服务器，但它具备了一个IM应有的全部功能。

#### 包括但不限于：

* [x] 发送消息（可定时的、可群发），包括：文本、图片、语音、视频、文件、位置、名片、撤回、转发、@、表情、对方正在输入、自定义消息等
* [x] 按需拉取离线消息，且没有消息数量/天数限制
* [x] 已读管理（对方是否已读、群内已读的成员）
* [ ] 音视频通话、IOS支持`Callkit`
* [x] 当用户不在线时，通过厂商推送（极光、腾讯、Mob）将消息推送给用户
* [x] 用户的每个会话都可以设置消息接收选项（接收、不接收、接收但不提醒）
* [x] 不限人数的群聊
* [x] 端对端加密


## xxim的背景

2022年初，我们公司的社交产品需要一个IM，但是我们不想使用第三方IM，所以我们自己开发了一个IM，但是我们发现开发一个IM并不容易，所以我们决定开源出来，让更多的人能够使用自己的IM。

* 服务端使用 Go 语言开发
    * 高性能
    * 简单语法，易于维护代码
    * 部署简单
    * 服务器资源占用少
* 客户端使用 flutter 开发
    * 跨平台、一套代码多端运行
    * 支持原生系统调用，性能强大
    * 界面美观、交互流畅

## xxim的设计原则

通过im服务器，我们希望解决以下问题：

* 聊天受监控
* 消息漫游天数有限制
* 群聊人数有限制
* 消息占用磁盘空间过大

## xxim-server 架构


## 点点star! ⭐

如果你喜欢或正在使用这个项目来学习或开始你的解决方案，请给它一个星。谢谢！

[![Star History Chart](https://api.star-history.com/svg?repos=cherish-chat/xxim-server&type=Date)](#xxim-server)
