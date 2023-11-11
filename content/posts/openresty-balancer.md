---
title: "使用balancer_by_lua实现动态转发"
subtitle: ""
date: 2023-11-10T13:13:00+08:00
lastmod: 2023-11-10T13:13:00+08:00
description: ""

tags: ["OpenResty"]
categories: ["OpenResty"]
---

## 0x01 需求

与第三方对接时，平台只能配置1个回调地址，但现在又有多个服务地址。根据平台回调功能，每个请求header中都会携带server(`server_id`)。围绕`server_id`实现特定转发功能。

## 0x02 实现

大致实现思路
1. `init_by_lua`阶段，使用`lua-resty-dns-client`读取`/etc/resolv.conf`配置，主要为nameserver和search参数
2. `rewrite_by_lua`阶段，通过header中的server获取对应转发服务的`server_id`
    - 服务完整dns记录格式 `zone{server_id}-{serviceName}.{search}参数`
    - 查询到对应A记录后，存储至ngx.shared.dnsCache中，同时设置`ngx.ctx.backend_ip`变量
3. `balancer_by_lua`阶段，从`ngx.ctx.backend_ip`变量获取ip，使用`set_current_peer`设置使用的后端服务器。

### 依赖
- OpenResty 1.21.4.3
- [lua-resty-dns-client](https://github.com/api7/lua-resty-dns-client)
    - [Penlight](https://github.com/lunarmodules/Penlight)

### 具体实现

- vhost.conf
```lua
lua_shared_dict resolvConf 32k;
lua_shared_dict dnsCache 1m;

init_by_lua_block {
    local cjson = require "cjson.safe"
    local dnsutils = require "resty.dns.utils"

    local config = ngx.shared.resolvConf
    local resolvConf = dnsutils.parseResolvConf()

    for name, value in pairs(resolvConf) do
        config:set(name, cjson.encode(value))
    end
}

upstream backend_svc {
    server 0.0.0.1 down; # 占位server

    balancer_by_lua_block {
        local balancer = require "ngx.balancer"
        balancer.set_timeouts(1, 0.5, 0.5)  -- 后端的连接、读、写超时时间
        balancer.set_more_tries(1)          -- 连接失败后最多重试1次
        local ip = ngx.ctx.backend_ip
        local ok, err = balancer.set_current_peer(ip, 8101)
        if not ok then
            ngx.log(ngx.ERR, "failed to set current_peer: ", err)
            return ngx.exit(500)
        end
    }

    keepalive 10;
}

server {
    listen       8101;
    server_name  _;

    location / {
        rewrite_by_lua_file '/rewrite_handle.lua';
        #默认值为0：重试次数不受限制
        proxy_next_upstream_tries 2;
        proxy_pass http://backend_svc;
    }
}
```

- `rewrite_handle.lua`
```lua
local cjson = require "cjson"
local resolver = require "resty.dns.resolver"
local nkeys = require "table.nkeys"
local json_decode = cjson.decode
local ngx = ngx
local tostring = tostring

local uri = ngx.var.uri
local method = ngx.req.get_method()
local resolvConf = ngx.shared.resolvConf
local dnsCache = ngx.shared.dnsCache
local ttl = 60

-- 获取resolvConf中search(即域名后缀)
local function getDomainSuffix()
    local domain_suffix = resolvConf:get("search")
    if domain_suffix then
        local t_domain_suffix = json_decode(domain_suffix)
        if nkeys(t_domain_suffix) > 0 then
            return t_domain_suffix[1]
        end
    end
end

local function getServiceIP(host)
    local ipAddr = dnsCache:get(host)
    if ipAddr then
        return ipAddr
    else
        local nameserver = resolvConf:get("nameserver")
        if nameserver then
            local r, err = resolver:new{
                nameservers = json_decode(nameserver),
                no_random = true -- always start with first nameserver
            }

            if not r then
                ngx.log(ngx.ERR, "failed to instantiate the resolver: ", err)
                return
            end

            local domain_suffix = getDomainSuffix()
            local fullHost = host .. "." .. domain_suffix

            local answers, err = r:query(fullHost, {
                qtype = r.TYPE_A
            })
            if not answers then
                ngx.log(ngx.ERR, "failed to query the DNS server: ", err)
                return
            end

            if answers.errcode then
                ngx.log(ngx.ERR, "server returned error code: ", answers.errcode, ": ", answers.errstr, ": ", fullHost)
                return
            end

            if nkeys(answers) > 0 then
                ipAddr = answers[1].address
                dnsCache:set(host, ipAddr, ttl)
                return ipAddr
            end

            return
        end
    end
end

if method == "POST" then
    local headers = ngx.req.get_headers()
    local serverId = headers["server"]
    if serverId then
        local serviceName = "default"
        if uri == "/mail" then
            serviceName = "mail"
        end

        local backend_addr = "zone" .. tostring(serverId) .. "-" .. serviceName
        local backend_ip = getServiceIP(backend_addr)
        if not backend_ip then
            return ngx.exit(502)
        end
        ngx.ctx.backend_ip = backend_ip
    else
        return ngx.exit(403)
    end
else
    return ngx.exit(405)
end
```

- 注意
    - `balancer.set_current_peer`：设置使用的后端服务器，必须是 IP 地址，不能是域名。
    - `balancer.set_more_tries` 与 nginx的`proxy_next_upstream_tries`互斥。
        - `proxy_next_upstream_tries`默认值为0(即无限重试)，`set_more_tries`不生效。
        - `proxy_next_upstream_tries`和`set_more_tries`同时配置都大于0时，以`set_more_tries`为准。
