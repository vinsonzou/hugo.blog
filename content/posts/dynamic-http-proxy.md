+++
description = ""
date = "2021-06-03T14:38:08+08:00"
title = "使用动态 DNS 来完成 HTTP 请求"
tags = ["OpenResty", "Nginx"]
categories = ["OpenResty"]

+++

内部访问外部http请求，通过拦截内部请求至此统一访问外部业务。

## 环境

- CentOS 7
- Nginx
    - lua-nginx-module
    - [lua-resty-dns](https://github.com/openresty/lua-resty-dns)
    - [lua-resty-lock](https://github.com/openresty/lua-resty-lock)
    - [lua-resty-http](https://github.com/ledgetech/lua-resty-http)

## 配置

**对应nginx配置**

```sh
lua_shared_dict dns_cache 5m;
lua_shared_dict my_locks 1m;

server {
    listen 80;
    server_name *.test.com;
    access_log logs/dyn_http.access.log default;

    location / {
        content_by_lua_file 'lua/dyn_http.lua';
    }
}
```

**dyn_http.lua**

```lua
local require = require
local ngx = ngx
local resolver = require "resty.dns.resolver"
local resty_lock = require "resty.lock"
local http = require "resty.http"
local cache = ngx.shared.dns_cache

local function fail(msg, err)
    ngx.log(ngx.ERR, msg, err)
    return {status=ngx.HTTP_SERVICE_UNAVAILABLE}
end

local function get_domain_ip_by_dns(domain)
    local dns = "119.29.29.29"

    local r, err = resolver:new{
        nameservers = {dns, {dns, 53} },
        retrans = 5,  -- 5 retransmissions on receive timeout
        timeout = 2000,  -- 2 sec
    }

    if not r then
        return fail("failed to instantiate the resolver: ", err)
    end

    local answers, err = r:query(domain)
    if not answers then
        return fail("failed to query the DNS server: ", err)
    end

    if answers.errcode then
        local err = answers.errcode .. ": " .. answers.errstr
        return fail("server returned error code: ", err)
    end

    for i, ans in ipairs(answers) do
      if ans.address then
        return ans.address
      end
    end

    return
end

local function get_domain_ip( domain )
    -- step 1:
    local ip, err = cache:get(domain)
    if ip then
        return ip
    end

    if err then
        return fail("failed to get key from shm: ", err)
    end

    -- cache miss!
    -- step 2:
    local lock, err = resty_lock:new("my_locks")
    if not lock then
        return fail("failed to create lock: ", err)
    end

    local elapsed, err = lock:lock(domain)
    if not elapsed then
        return fail("failed to acquire the lock: ", err)
    end

    -- lock successfully acquired!

    -- step 3:
    -- someone might have already put the value into the cache
    -- so we check it here again:
    ip, err = cache:get(domain)
    if ip then
        local ok, err = lock:unlock()
        if not ok then
            return fail("failed to unlock: ", err)
        end

        return ip
    end

    --- step 4:
    local ip = get_domain_ip_by_dns(domain)
    if not ip then
        local ok, err = lock:unlock()
        if not ok then
            return fail("failed to unlock: ", err)
        end

        -- FIXME: we should handle the backend miss more carefully
        -- here, like inserting a stub value into the cache.

        return
    end

    -- update the shm cache with the newly fetched value
    local ok, err = cache:set(domain, ip, 120)
    if not ok then
        local ok, err = lock:unlock()
        if not ok then
            return fail("failed to unlock: ", err)
        end

        return fail("failed to update shm cache: ", err)
    end

    local ok, err = lock:unlock()
    if not ok then
        return fail("failed to unlock: ", err)
    end

    return ip
end

local function http_request_with_dns(domain, params)
    -- get domain's ip
    local domain_ip, err = get_domain_ip(domain)
    if not domain_ip then
        ngx.log(ngx.ERR, "get the domain[", domain ,"] ip by dns failed:", err)
        return {status=ngx.HTTP_SERVICE_UNAVAILABLE}
    end

    -- http request
    local httpc = http.new()
    --local uri = ngx.var.uri
    --local url = string.format("http://%s/%s",domain_ip, uri)
    local request_uri = ngx.var.request_uri
    local url = string.format("http://%s%s",domain_ip, request_uri)

    local res, err = httpc:request_uri(url, params)

    if err then
        return {status=ngx.HTTP_SERVICE_UNAVAILABLE}
    end

    -- httpc:request_uri 内部已经调用了keepalive，默认支持长连接
    -- httpc:set_keepalive(1000, 100)
    -- 开启长连接时，lua_code_cache 必须要设置为 on ，参考https://groups.google.com/forum/#!topic/openresty/gLXnpR_EKig
    return res
end

local params = {}
local method = ngx.req.get_method()
local headers = ngx.req.get_headers()
local domain = headers["Host"]

params.method = method
params.path = ngx.var.uri
params.headers = headers

if method == "GET" or method == "HEAD" then
    params.query = ngx.req.get_uri_args()
elseif method == "POST" then
    ngx.req.read_body()
    local body = ngx.req.get_body_data()
    params.body = body
end

local res = http_request_with_dns(domain, params)

if res.status == 301 or res.status == 302 then
    local url = res.headers["Location"]
    return ngx.redirect(url)
end

if res.status == 200 then
    for k,v in pairs(res.headers) do
        ngx.header[k] = v
    end
    ngx.say(res.body)
else
    ngx.exit(res.status)
end
```

## 参考

- https://moonbingbing.gitbooks.io/openresty-best-practices/content/dns/use_dynamic_dns.html
