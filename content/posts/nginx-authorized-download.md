+++
description = ""
date = "2021-06-11T11:18:08+08:00"
title = "使用Nginx实现授权下载"
tags = ["OpenResty"]

+++

使用Nginx实现授权下载有两种方式，一种是nginx自带 `--with-http_secure_link_module`，另外一种是lua自定义实现。

## 方法一

**1.1 安装**

编译时添加此参数 `--with-http_secure_link_module`

**1.2 配置**

```sh
server {
    listen 80;
    server_name download.test.com;
    root /data/wwwroot;
    access_log logs/download.access.log main;

    location / {
        secure_link $arg_st,$arg_e;
        secure_link_md5 FIJaVSEV23$uri$arg_e;

        if ($secure_link = "") {
            return 403;
        }

        if ($secure_link = "0") {
            return 410;
        }
    }
}
```

**1.3 使用**

```sh
# shell脚本示例如下
secret="FIJaVSEV23"     # 见前面secure_link_md5配置
uri="/待下载文件路径"
datetime=$(date -d'1day' +'%Y-%m-%d %H:%M:%S')
etime=$(date +%s -d"${datetime}")
token=$(echo -n "${secret}${uri}${etime}" | openssl md5 -binary | openssl base64 | tr +/ -_ | tr -d =)

# 下载链接
dlurl="dowload.test.com${uri}?st=${token}&e=${etime}"
```

## 方法二

**2.1 环境**

- CentOS 7
- Nginx
    - lua-nginx-module

**2.2 配置**

- 授权有效期300秒
- apk和通用文件下载，使用参数形式认证
- 通过plist形式安装的ipa不能使用任何参数，参数都固化在uri中

```sh
server {
    listen 80;
    server_name download.test.com;
    root /data/wwwroot;
    access_log logs/download.access.log main;

    location / {
        access_by_lua_file lua/download_access.lua;
    }
}
```

**download_access.lua**

{{< gist vinsonzou 4507dca6d58d933eaf38a1079850d78b >}}

**2.3 使用**

```sh
# shell示例

key="abc"
ts="1623402947"
sign=$(echo -n '${ts}#test#${key}' | md5sum)

apk和通用下载链接格式: http://download.test.com/xxx.apk?sign=${sign}&ts=${ts}
plist中ipa下载链接格式: http://download.test.com/xxx.ipa/${sign}/${ts}
```
