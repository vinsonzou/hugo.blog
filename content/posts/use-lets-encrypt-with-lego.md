---
title: "使用lego申请Let's Encrypt通配符证书"
subtitle: ""
date: 2022-01-22T15:20:03+08:00
lastmod: 2022-02-10T16:30:00+08:00
description: ""

tags: ["安全"]
categories: ["安全"]
---

当前使用Let's Encrypt颁发的证书，原先使用python的certbot，最近发现有个go版更简单无环境依赖的[lego](https://github.com/go-acme/lego)，目前自己使用的国内DNS都支持，如阿里云DNS、DNSPOD、腾讯云DNS。

以阿里云DNS为例，使用示例如下

```sh
ALICLOUD_ACCESS_KEY=abcdefghijklmnopqrstuvwx \
ALICLOUD_SECRET_KEY=your-secret-key \
./lego --email myemail@example.com --dns alidns --domains *.example.com run
```
> 默认返回为ec256证书，可以添加参数-k rsa2048调整

返回如下

![](/img/lego01.png)

证书目录如下

![](/img/lego02.png)

DNSPOD DNS示例如下：

```sh
DNSPOD_API_KEY=xxxxxx \
lego --email myemail@example.com --dns dnspod --domains *.example.org run
```

腾讯云DNS示例如下：
```sh
TENCENTCLOUD_SECRET_ID=abcdefghijklmnopqrstuvwx \
TENCENTCLOUD_SECRET_KEY=your-secret-key \
lego --email myemail@example.com --dns tencentcloud --domains my.example.org run
```

> Ps: 腾讯云相同的key，使用certbot可以申请证书，使用lego就不行，一直报错
> `acme: error presenting token: tencentcloud: create record failed: [TencentCloudSDKError] Code=InvalidParameter.DomainInvalid, Message=域名不正确，请输入主域名，如 dnspod.cn。, RequestId=96b61fd1-a225-4455-821f-63101f59a117`
> 无奈腾讯云域名就只能使用dnspod token来使用lego了。

更多DNS厂商使用案例，请参考[官网](https://go-acme.github.io/lego/dns/)

## 更新记录

- 2022-02-10: 更新腾讯云DNS插件无法使用，用DNSPOD插件替代解决
