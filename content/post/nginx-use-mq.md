+++
description = ""
date = "2021-06-03T15:08:08+08:00"
title = "lua发布消息至RocketMQ解耦"
tags = ["OpenResty", "Nginx", "RocketMQ"]

+++

广告点击回传数据入消息队列，避免后端处理不过来，先存入消息队列削峰。

## 环境

- CentOS 7
- Nginx
    - lua-nginx-module
    - [lua-resty-core](https://github.com/openresty/lua-resty-core)
    - [lua-resty-http](https://github.com/ledgetech/lua-resty-http)
- 依赖服务
    - 阿里云RocketMQ

## 配置

**对应nginx location配置**

```sh
location / {
    access_by_lua_file lua/mq/mq.lua;
}
```

**custom/mq.lua**

```lua
return {
    AKId = "xxx",          -- RocketMQ的AccessKeyId
    AKSecret = "xxx",      -- RocketMQ的AccessKeySecret
    topic_name = "xxx",    -- RocketMQ的topic
    instance_id = "xxx"    -- RocketMQ的instance_id
}
```


**mq.lua**

```lua
local require = require
local ngx = ngx
local config = require("custom.mq.config")
local http = require "resty.http"
local cjson = require "cjson.safe"
local ngx_re = require "ngx.re"
local hmac_sha1 = ngx.hmac_sha1
local encode_base64 = ngx.encode_base64
local json_encode = cjson.encode
local re_find = ngx.re.find

local function get_headers(mq_uri)
    local HTTP_METHOD = "POST"
    local CONTENT_TYPE = "text/xml;charset=UTF-8"
    local HTTP_DATE = ngx.http_time(ngx.time())
    local MQVersion = "2015-06-06"
    local CanonicalizedResource = mq_uri
    local string_to_sign = HTTP_METHOD .. "\n\n" .. CONTENT_TYPE .. "\n" .. HTTP_DATE .. "\nx-mq-version:" .. MQVersion .. "\n" .. CanonicalizedResource
    local Signature = encode_base64(hmac_sha1(config.AKSecret, string_to_sign))
    local Authorization = "MQ " .. config.AKId .. ":" .. Signature

    local headers = {
        Authorization = Authorization,
        Date = HTTP_DATE,
        Host = "xxx.cn-beijing-internal.aliyuncs.com",  -- 配置阿里云RocketMQ内部地址
        ["x-mq-version"] = MQVersion,
        ["Content-Type"] = CONTENT_TYPE
    }

    return headers
end

local function mq_publish(data)
    -- 配置mq的topic和instance_id信息
    local mq_uri = "/topics/ad_click/messages?ns=MQ_xxxxxxxx"
    local headers = get_headers(mq_uri)
    local msg = '<?xml version="1.0" encoding="utf-8"?><Message xmlns="http://mq.aliyuncs.com/doc/v1/"><MessageBody>' .. data .. '</MessageBody></Message>'
    headers["Content-Length"] = #msg

    local httpc = http.new()
    -- 暂时写死的mq的地址
    local res = httpc:request_uri("http://100.100.18.17" .. mq_uri, {
        method = "POST",
        body = msg,
        headers = headers,
        keepalive_timeout = 60,
        keepalive_pool = 20
    })
    if res.status == 201 then
        return true
    else
        return false
    end
end

local method = ngx.req.get_method()
local uri = ngx.var.uri

-- 广告点击uri正则，按需定制
if re_find(uri, [[^/ad/click/[a-zA-Z0-9_]+/[a-zA-Z0-9]+$]], "jo") then
    local res, err = ngx_re.split(uri, "/")
    local game = res[5]
    local channel = res[6]

    if err then
        return
    end

    if channel == "jfq" then
        return
    end

    local args
    if method == "POST" then
        ngx.req.read_body()
        args, err = ngx.req.get_post_args()
    else
        args, err = ngx.req.get_uri_args()
    end

    local data = {game=game,channel=channel,params=args}
    local data_str = json_encode(data)
    local msg = encode_base64(data_str)
    local mq_status = mq_publish(msg)

    if mq_status then
        local t
        if channel == "wechat" or channel == "gdt" or channel == "newgdt" then
            t = {ret=0,msg=""}
        else
            t = {message="",data="",code=200,success=true}
        end

        ngx.header.content_type = "application/json;charset=UTF-8"
        ngx.say(json_encode(t))
        ngx.exit(200)
    end
end
```
