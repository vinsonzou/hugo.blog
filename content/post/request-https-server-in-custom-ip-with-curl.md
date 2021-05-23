+++
date = "2018-06-06T10:00:00+08:00"
draft = false
title = "使用curl请求https时指定IP"
tags = ["curl"]
+++

一般使用curl请求自定义IP地址并且指定HOST的话可以这样。

```sh
curl -H 'Host: ops.m114.org' http://127.0.0.1
```

但是如果你需要请求的地址是HTTPS就不行了

```sh
$ curl -H 'Host: ops.m114.org' https://127.0.0.1/
curl: (51) Unable to communicate securely with peer: requested domain name does not match the server's certificate.
```

因为IP绝大多数情况下无法通过域名证书验证，还好curl中有`--resolv`参数可以让我们方便的指定域名的解析

```sh
# --resolv参数形式
--resolv host:port:address
# 示例
curl --resolv ops.m114.org:443:127.0.0.1 https://ops.m114.org
```

Ps:

小编经常使用的CentOS6的自带curl就不支持此参数，幸运的是CentOS7已经支持。
