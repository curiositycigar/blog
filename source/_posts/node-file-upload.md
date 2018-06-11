---
title: node实现大文件上传
date: 2018-05-08 16:49:18
description: nodeJs大文件上传, 支持断点续传， 秒传
categories: 后端
tags: node
---
# node大文件上传

## 文件上传简介

文件上传仅在http请求 `content-type` 为 `multipart/form-data` 时支持。


## 实现原理

### 上传

> 计算文件md5值，用md5值对文件进行唯一认证。
> 利用filereader API对文件进行分片，并单独上传每个分片（分片附带md5值）。
> 服务端接收每个文件分片，并在所有分片上传完成后合并分片。

### 断点续传

> 分片上传将文件分为多个单独的小文件，所以当上传从某一位置终止时，服务端可以判断上次断开位置，并通知客户端。

### 秒传

> 利用md5值的近似唯一性，服务端判断如果md5值存在，就认为文件已经上传，返回上传成功（秒传）
> tip：md5值并不唯一。

## 客户端依赖
`spark-md5` js MD5计算插件
`axios` ajax 请求

## 后端依赖

`koa2` `koa-router` `koa-static` `koa-body`

## 项目地址

[点击此处](https://github.com/curiositycigar/nodeUpload)