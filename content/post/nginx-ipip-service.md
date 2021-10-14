+++
description = ""
date = "2021-06-03T13:08:08+08:00"
title = "Nginx实现ip查询服务"
tags = ["OpenResty", "Nginx"]

+++

实现内部IP查询服务
- 查询当前公网IP
- 查询当前公网IP地理信息，支持IPv4/IPv6
- 查询指定IP的地理信息，支持IPv4/IPv6

## 环境

- CentOS 7
- Nginx
    - lua-nginx-module
    - [lua-resty-ipmatcher](https://github.com/api7/lua-resty-ipmatcher)
    - [lua-resty-ip2region](https://github.com/shixinke/lua-resty-ip2region)
    - [lua-resty-maxminddb](https://github.com/anjia0532/lua-resty-maxminddb)
- IP库
    - [ip2region ipv4库](https://github.com/lionsoul2014/ip2region/blob/master/data/ip2region.db)
    - [maxminddb 免费ipv6库](https://dev.maxmind.com/geoip/geolite2-free-geolocation-data)

## 配置

```lua
lua_shared_dict ip_data 10m;

server {
    server_name ip.test.com;
    access_log logs/ip.access.log main buffer=32k flush=10s;
    error_log logs/ip.error.log;

    location / {
        default_type 'text/plain';
        access_by_lua_block {
            local remote_addr = ngx.var.remote_addr
            ngx.print(remote_addr)
        }
    }

    location = /ip {
        default_type application/json;
        content_by_lua_block {
            local require = require
            local ngx = ngx
            local cjson = require "cjson.safe"
            local json_encode = cjson.encode
            local ip2region = require "resty.ip2region"
            local ipmatcher = require "resty.ipmatcher"
            local geo = require "resty.maxminddb"
            local location = ip2region.new({
                file = "/data/nginx/lua/ip/ip2region.db",
                dict = "ip_data",
                mode = "memory" -- maybe memory,binary or btree
            })
            if not geo.initted() then
                geo.init("/data/nginx/lua/ip/GeoLite2-City.mmdb")
            end

            local remote_addr = ngx.var.arg_ip or ngx.var.remote_addr

            local ip_detail = {
                ip=remote_addr,
                country=0,
                province=0,
                city=0,
                isp=0,
            }
            local is_ipv4 = ipmatcher.parse_ipv4(remote_addr)
            if is_ipv4 then
                local data, err = location:search(remote_addr)
                if data then
                    ip_detail["country"] = data["country"]
                    ip_detail["province"] = data["province"]
                    ip_detail["city"] = data["city"]
                    ip_detail["isp"] = data["isp"]
                    ngx.say(json_encode(ip_detail))
                else
                    ngx.say(err)
                end
            else
                local is_ipv6 = ipmatcher.parse_ipv6(remote_addr)
                if is_ipv6 then
                    local data, err = geo.lookup(remote_addr)
                    if data then
                        ip_detail["country"] = data["country"]["names"]["zh-CN"]
                        ip_detail["continent"] = data["continent"]["names"]["zh-CN"]
                        if data["subdivisions"] then
                            local subdivisions = data["subdivisions"][1]["names"]
                            local province_cn = subdivisions["zh-CN"]
                            if province_cn then
                                ip_detail["province"] = province_cn
                            else
                                ip_detail["province"] = subdivisions["en"]
                            end
                        end

                        if data["city"] then
                            local city = data["city"]["names"]
                            local city_cn = city["zh-CN"]
                            if city_cn then
                                ip_detail["city"] = city_cn
                            else
                                ip_detail["city"] = city["en"]
                            end
                        end
                        ngx.say(json_encode(ip_detail))
                    end
                end
            end
        }
    }
}
```

## 使用示例

```sh
$ curl ip.test.com
180.167.11.11

$ curl ip.test.com/ip
{"ip":"180.167.11.11","country":"中国","province":"上海","city":"上海","isp":"电信"}

$ curl 'ip.test.com/ip?ip=1.1.1.1'
{"isp":"CloudFlare","ip":"1.1.1.1","country":"澳大利亚","province":"0","city":"0"}

$ curl 'ipip.test.com/ip?ip=240b:4000:f10::e3'
{"ip":"240b:4000:f10::e3","country":"新加坡","province":0,"city":0,"isp":0,"continent":"亚洲"}
```
