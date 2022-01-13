---
title: "nginx 下载文件重命名"
subtitle: ""
date: 2022-01-13T13:40:03+08:00
lastmod: 2022-01-13T13:40:03+08:00
description: ""

tags: ["nginx"]
categories: ["nginx"]
---

当前Nginx配置

```
location /download/ {
    if ($arg_file) {
        add_header Content-Disposition "attachment;filename*=utf-8'zh_cn'$arg_file";
    }
}
```

> Chrome、Firefox正常，但Safari下载文件名为当前域名

调整后Nginx配置

```
location /download/ {
    if ($arg_file) {
        add_header Content-Disposition "attachment;filename*=UTF-8''$arg_file";
    }
}
```

> 调整后，Safari正常了(中文文件名正常)。
