---
# layout: post
title: "WebRTC（一）简介和基本原理"
subtitle: ""
date: 2023-02-16
author: "汪洋龙"
categories: [Tech]
image: "img/coffee-bg.jpg"
description: "webRTC"
tags:
  - webrtc
---

# WebRTC

WebRTC（Web Real-Time Communication），是一个支持网页浏览器进行实时语音对话或视频对话的 API。它于 2011 年 6 月 1 日开源并在 `Google`、`Mozilla`、`Opera` 支持下被纳入万维网联盟的 W3C 推荐标准。

## 重要 API

WebRTC 原生 APIs 文件是基于 WebRTC 规格书撰写而成，这些 API 可分为 `Network Stream API`, `RTCPeerConnection`, `Peer-to-peer Data` API 三类。

### Network Stream API

- MediaStream：`MediaStream` 用来表示一个媒体数据流
- MediaStreamTrack 在浏览器中表示一个媒体源

### RTCPeerConnection

- RTCPeerConnection：一个 `RTCPeerConnection` 对象允许用户在两个浏览器之间直接通讯
- RTCIceCandidate：表示一个 `ICE` 协议的候选者
- RTCIceServer：表示一个 `ICE Server`

### Peer-to-peer Data

- DataChannel：数据通道（`DataChannel`）接口表示一个在两个节点之间的双向的数据通道

## WebRTC 通话原理

两个不同网络环境的（具备摄像头/麦克风多媒体设备的）浏览器，要实现点对点实时音视频对话所要考虑的问题。

### 1. 媒体协商【sdp】

彼此了解对方支持的媒体格式，eg：

- Peer-A：支持 VP8 + H264
- Peer-B：支持 VP9 + H264

如果要保证两端都可以正确的编解码，最简单方法就是取交集 `H264`

有一个专门的协议，称为 `Session Description Protocol（SDP）`,可用于描述上述这类信息，在 WebRTC 中，参与视频通讯的**双方必须先交换 SDP 信息**，在交换 SDP 的过程，也称为 `媒体协商`。

### 2. 网络协商【candidate】

彼此了解对方的网络情况，找到通讯链路

- 获取外网 IP 地址映射
- 通过信令服务器（signal server）交换网络信息

**理想情况** 每个电脑存在私有公网 IP，可以直接进行点对点连接。

**实际情况** 每个电脑都在某个局域网中，需要 `NAT（NetWork Address Translation，网络地址转换）`

`STUN（Session Traversal Utilities for NAT，NAT会话穿越应用程序）`是一种网络协议，它允许位于 NAT（或多重 NAT）后的客户端找出自己的公网地址，查出自己位于哪种类型的 NAT 之后以及 NAT 为某一个本地端口所绑定的 Internet 端端口。这些信息被用来在两个同时处于 NAT 路由器之后的主机之间创建 UDP 通信。

`TURN（Traversal Using Relays around NAT）`是 `STUN/RFC5389` 的一个拓展，主要添加了 `Relay` 功能。如果终端在 NAT 之后，在特定的情况下，有可能使得终端无法和其对等端（peer）进行直接的通信，这就需要公网的服务器作为一个中继，对来往的数据进行转发，这个转发的协议就被定义为 TURN。

`STUN` 和 `TURN` 可以用 [coturn](https://github.com/coturn/coturn) 搭建。

`candidate` 用来描述网络信息

### 3. 信令服务器

拿到了两个客户端的`媒体信息`和`网络信息`，需要一个信令服务器（signal server）转发彼此的媒体信息和网络信息。
