---
title: HTTP 库 - Android、iOS 以及其他
date: "2021-10-17T22:12:03.284Z"
description: "HTTP 库简述"
---

最近由于工作原因，重新温习了 Android 和 iOS 系统中的 HTTP 库。除去系统原生 API 之外，还有一些社区封装的库。比较流行的莫属 OkHttp 和 AFNetworking。下文会对这两个库做一些梳理。

## OkHttp

OkHttp 应该是 Android 最流行的开源 HTTP 库。根据官方文档的描述，OkHttp 支持以下特性：

- HTTP/2。允许所有对同一个 host 的请求使用同一个 socket。
- 连接池。可以在不能使用 HTTP/2 的情况下降低请求延时。
- 透明压缩。OkHttp 会自动加入 gzip 请求头 `Accept-Encoding:gzip`。所以，当返回的数据带有 gzip 响应头 `Content-Encoding=gzip` 时，OkHttp 会自动解压数据。
- 响应缓存。支持在不消耗网络资源的情况下处理重复的请求。

## AFNetworking

AFNetworking 应该是 iOS 最流行的开源 HTTP 库。我们可以把 Alamofire 也作为 AFNetworking 的延伸版。