---
title: whistle网页抓包
date: 2022-11-28 20:26:46
categories:
- 开发辅助工具
tags:
- 抓包
- 网络
---
# Whistle抓包过程

## 电脑抓包

新版的 whistle 支持三种等价的命令 whistle、w2、wproxy

  启动 whistle：w2 start 

  重启 whistle：w2 restart

  停止 whistle：w2 stop

  默认端口 8899，启动时可以通过 w2 start -p newPort 自定义端口

  每次使用时都要重新启动 Whistle

\4. 打开 Whistle 监控界面

  http://192.168.63.183:8899

访问局域网的这个页面

## 手机抓包

#### (a) 电脑分享生成Wi-Fi，设备连接并设置代理。

以下以手机为例进行说明，【Wi-Fi连接】-【代理-手动】-【配置服务器&端口】
 **iPhone：**【http代理-配置代理-手动】-【服务器（电脑ip）&端口（8899）】
 **huawei p30：**【Wi-Fi连接后，显示高级选项】-【代理-手动】-【服务器（电脑ip）&端口（8899）】

#### (b) 安装证书

点击抓包网页上方的`HTTPS`生成一个二维码，扫码安装到手机上

#### (c) 访问相关内容

点击抓包网页，点击左侧的`Network`，手机访问相关内容即可抓包

![img](https:////upload-images.jianshu.io/upload_images/7917329-9e7f4f63145b9f4d.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)



## 本地测试线上

使用whistle的rules配置代理，访问线上的代理转向本地，就可以实现本地开发同时抓取线上的请求去进行本地开发测试

