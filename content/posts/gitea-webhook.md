+++
description = ""
date = "2021-06-03T10:08:08+08:00"
title = "Gogs/Gitea webhook支持"
tags = ["OpenResty", "Nginx", "Gogs", "Gitea"]
categories = ["OpenResty"]

+++

实现需求为git提交后，自动同步至webhook所在机器的指定目录。也可以支持github的webhook，也能根据不同条件触发不同的webhook，如CI/CD。

## 环境

- CentOS 7
- Nginx
    - ngx_lua
    - [lua-resty-shell](https://github.com/juce/lua-resty-shell)
    - [lua-resty-hmac](https://github.com/jkeys089/lua-resty-hmac)

## 配置

**对应nginx location配置**

```sh
location = /webhook/deploy {
    content_by_lua_file 'lua/webhook/deploy.lua';
}
```

**deploy.lua**

```lua
-- Gogs Version: >= v0.10
local require = require
local ngx = ngx
local cjson = require("cjson.safe")
local shell = require("resty.shell")
local hmac = require("resty.hmac")
local json_encode = cjson.encode
local json_decode = cjson.decode

-- deploy config
local config = {
    timeout = 30 * 1000,  -- 30s
    socket = "unix:/data/nginx/utils/shell.sock", -- fixed for CentOS7
    repo_a = {
        secret = "f659f833-9dc8-4c7d-ace9-4e2f8929759d",
        deploy_path = "/data/",
    },
    repo_b = {
        secret = "27a10815-01c3-4f7a-80dc-d5a2c60ca155",
        deploy_path = "/data/",
    },
}

local function say_json(msg)
    ngx.say(json_encode(msg))
end

local function say_error(retval, errmsg, shell_out, shell_err)
    say_json({result = retval, msg = errmsg, shell_out = shell_out, shell_err = shell_err})
end

local function say_ok(shell_out, shell_err)
    say_json({result = 0, msg = "ok", shell_out = shell_out, shell_err = shell_err})
end

ngx.header.content_type = "application/json"

-- only accept POST request
local method = ngx.req.get_method()
local headers = ngx.req.get_headers()
local x_gogs_signature = headers["X-Gogs-Signature"]

if method == "POST" then
    ngx.req.read_body()
    local body = ngx.req.get_body_data()
    local data = json_decode(body)

    if not x_gogs_signature then
        say_error(-1, "invalid operation")
    else
        local repo = data["repository"]["name"]
        local secret = config[repo]["secret"]

        local hmac_sha256 = hmac:new(secret, hmac.ALGOS.SHA256)
        local signature = hmac_sha256:final(body, true)

        if x_gogs_signature == signature then
            local shell_cmd = "cd "..config[repo]["deploy_path"]..repo.."; git reset --hard origin/master; git clean -f; git pull"
            local status, out, err = shell.execute(shell_cmd, {timeout = config.timeout})

            -- print result
            if status ~= 0 then
                say_error(status, "execute failed", out, err)
            else
                say_ok(out, err)
            end
        else
            say_error(-1, "token incorrect")
        end
    end
else
    say_error(-1, "invalid request type")
end
```
