+++
description = ""
date = "2021-06-02T10:08:08+08:00"
title = "Nginx支持WebP"
tags = ["OpenResty", "WebP"]
categories = ["OpenResty"]

+++

WebP格式，Google开发的一种旨在加快图片加载速度的图片格式。图片压缩体积大约只有JPEG的2/3，并能节省大量的服务器带宽资源和数据空间。国外的有 Google（自家的东西肯定要用啦，Chrome Store 甚至已全站使用 WebP）、Facebook 和 ebay，国内的有淘宝、腾讯和美团等。

客户端支持情况:

- Google Chrome 23 起开始支持 WebP（最初发布于2012年11月）
- Google 的安卓浏览器从 4.2 版本起开始官方支持 WebP（最初发布于2012年11月），4 版本起开始部分支持
- Google Chrome安卓版从 Chrome 50 起开始支持 Webp
- Opera 12.1 开始支持 WebP（最初发布于2012年11月）
- Edge 18 开始支持 (发布于2018年10月)
- Firefox 65 支持 (发布于2019年1月)

nginx无缝切换至WebP思路就是：用户访问一张图片(主要为png/jpg格式)，nginx收到请求判断浏览器是否支持WebP，如果支持则返回WebP，不支持则返回原图，达到减少带宽资源的目的。

## 环境

- CentOS 7
- [libwebp-1.2.0](https://storage.googleapis.com/downloads.webmproject.org/releases/webp/libwebp-1.2.0-linux-x86-64.tar.gz)
- Nginx
    - ngx_lua
    - [lua-resty-shell](https://github.com/juce/lua-resty-shell)

## 安装部署

**cwebp依赖lib**

```sh
yum -y install libGL libX11 libXi
```

## 配置

**mime.types新增webp格式支持**

```sh
image/webp webp;
```

**图片对应nginx location配置**

```sh
location ~ ^/(images|static)/ {
	rewrite_by_lua_file "/data/nginx/lua/webp.lua";
}
```

**webp.lua**

```lua
local require = require
local ngx = ngx
local shell = require("resty.shell")
local re_find = ngx.re.find

local config = {
    timeout = 30 * 1000,  -- 30s
    socket = "unix:/data/nginx/utils/shell.sock", -- fixed for CentOS7
}

local function file_exists(name)
    local f=io.open(name,"r")
    if f~=nil then io.close(f) return true else return false end
end

local uri = ngx.var.uri

if re_find(uri, [[.webp$]], "jo") then
    return
elseif re_find(uri, [[.(png|jpg)$]], "joi") then
    local originalFile = ngx.var.request_filename
    local webpFile = originalFile .. ".webp"

    local accept_header = ngx.req.get_headers()["Accept"]
    if accept_header and re_find(accept_header, [[image/webp]], "jo") then
        if not file_exists(originalFile) then
            ngx.exit(404)
            return
        end

        local webp_uri = uri .. ".webp"
        if file_exists(webpFile) then
            return ngx.exec(webp_uri)
        else
            local shell_cmd = "cwebp -quiet -q 75 "..originalFile.." -o "..webpFile
            local status, out, err = shell.execute(shell_cmd, {timeout = config.timeout, socket = config.socket})
            return ngx.exec(webp_uri)
        end
    end
end
```

**特别说明**

- 已存在转换后的WebP图片时，更新原始png/jpg图片时，WebP图片不会更新
