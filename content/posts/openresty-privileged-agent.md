+++
description = ""
date = "2021-10-08T11:13:08+08:00"
title = "使用OpenResty reload ipsec服务"
tags = ["OpenResty"]

+++

## 背景介绍

- Debian 11
- OpenResty 1.19.9.1

默认情况下，Nginx/OpenResty会启动一个root权限运行的master进程，之后再用指定的普通用户权限启动对应的worker，因要root权限才能操作ipsec服务，为了解决普通worker的提权问题，OpenResty提供了一个privileged agent来对这些提权操作进行处理，本文就是利用privileged agent来实现ipsec服务的重启操作。

## 基本思路

在OpenResty的 `init_by_lua_block` 阶段启动privileged agent，之后在 `init_worker_by_lua_block` 阶段针对privileged agent设置其具体需要执行的操作，最终在 `content_by_lua_block` 中通过改变共享内存的 `lua_shared_dict` 字段的内容触发对应的reload操作。

## 对应配置

```sh
root@debian:~# cat /etc/openresty/nginx.conf
...
user app; #普通用户启动worker
...
http {
    ...
    lua_shared_dict reload_status 1m; #共享内存dict

	init_by_lua_block {
	    local process = require "ngx.process"
	
	    -- enables privileged agent process
	    local ok, err = process.enable_privileged_agent()
	    if not ok then
	        ngx.log(ngx.ERR, "enables privileged agent failed error:", err)
	    end
	
	    -- output process type
	    ngx.log(ngx.INFO, "process type: ", process.type())
	}
	
	init_worker_by_lua_block {
	    local process = require "ngx.process"
	    local shell = require "resty.shell"
	
	    local timeout = 3000  -- ms
	    local max_size = 4096  -- byte
	
	    local function ipsec_reload(premature)
	        local reload_status = ngx.shared.reload_status
	        local value, flags = reload_status:get("ipsec")
	        if value == 1 then
	            ngx.log(ngx.ALERT, "ipsec reloading ")
	            local shell_cmd = "/usr/sbin/ipsec reload"
	            local ok, stdout, stderr, reason, status = shell.run(shell_cmd, nil, timeout, max_size)
	            local value, flags = reload_status:set("ipsec",0)
	        end
	    end
	
	    if process.type() == "privileged agent" then
	        local ok, err = ngx.timer.every(60, ipsec_reload)
	        if not ok then
	            ngx.log(ngx.ERR, "ipsec info: ", err)
	        end
	    end
	}

    server {
        listen 80;
        server_name t.test.com;

    	location /t {
    		#如果是生产环境，需要加上IP白名单或mTLS认证来提高安全性
    	    content_by_lua_block {
    	        local reload_s = ngx.shared.reload_status
    	        reload_s:set("ipsec", 1)
    	        ngx.say("reload sucessful")
    	    }
    	}
	}
```

## 总结

通过上面的操作可以很方便的给OpenResty进行扩展执行任何root权限执行的功能，可以作为运维维护的辅助利器，但切记做好安全防护哟。
