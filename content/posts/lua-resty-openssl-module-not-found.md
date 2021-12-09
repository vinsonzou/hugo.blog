---
title: "lua-resty-openssl module not found"
subtitle: ""
date: 2021-12-07T20:08:13+08:00
lastmod: 2021-12-07T20:08:13+08:00
description: ""

tags: ["Nginx","OpenResty"]
categories: ["OpenResty"]
---

## 环境

- OS: CentOS 8.4
- OpenResty: 1.19.9.1(手动编译)
- OpenResty prefix: /app/nginx
- lua_package_path `"$prefix/lualib/?.lua;;";`

## 报错如下

```
2021/12/07 18:56:11 [error] 2383822#0: *1 lua entry thread aborted: runtime error: /app/nginx//lualib/resty/openssl/pkey.lua:13: module 'resty.openssl.include.x509' not found:
	no field package.preload['resty.openssl.include.x509']
	no file '/app/nginx//lualib/resty/openssl/include/x509.lua'
	no file './resty/openssl/include/x509.lua'
	no file '/app/nginx/luajit/share/luajit-2.1.0-beta3/resty/openssl/include/x509.lua'
	no file '/usr/local/share/lua/5.1/resty/openssl/include/x509.lua'
	no file '/usr/local/share/lua/5.1/resty/openssl/include/x509/init.lua'
	no file '/app/nginx/luajit/share/lua/5.1/resty/openssl/include/x509.lua'
	no file '/app/nginx/luajit/share/lua/5.1/resty/openssl/include/x509/init.lua'
```

## 其他环境参考

诡异的是，相同代码相同的lua_package_path，在Mac上(通过brew安装)和Debian 11上(通过apt安装)OpenResty却一切正常。

- Mac的package_path列表如下
```
/opt/homebrew/Cellar/openresty/1.19.9.1_2/nginx//lualib/?.lua;
/opt/homebrew/Cellar/openresty/1.19.9.1_2/site/lualib/?.ljbc;
/opt/homebrew/Cellar/openresty/1.19.9.1_2/site/lualib/?/init.ljbc;
/opt/homebrew/Cellar/openresty/1.19.9.1_2/lualib/?.ljbc;
/opt/homebrew/Cellar/openresty/1.19.9.1_2/lualib/?/init.ljbc;
/opt/homebrew/Cellar/openresty/1.19.9.1_2/site/lualib/?.lua;
/opt/homebrew/Cellar/openresty/1.19.9.1_2/site/lualib/?/init.lua;
/opt/homebrew/Cellar/openresty/1.19.9.1_2/lualib/?.lua;
/opt/homebrew/Cellar/openresty/1.19.9.1_2/lualib/?/init.lua;
./?.lua;
/opt/homebrew/Cellar/openresty/1.19.9.1_2/luajit/share/luajit-2.1.0-beta3/?.lua;
/usr/local/share/lua/5.1/?.lua;
/usr/local/share/lua/5.1/?/init.lua;
/opt/homebrew/Cellar/openresty/1.19.9.1_2/luajit/share/lua/5.1/?.lua;
/opt/homebrew/Cellar/openresty/1.19.9.1_2/luajit/share/lua/5.1/?/init.lua;
```

- Mac的configure参数
```
nginx version: openresty/1.19.9.1
built by clang 12.0.5 (clang-1205.0.22.9)
built with OpenSSL 1.1.1k  25 Mar 2021 (running with OpenSSL 1.1.1l  24 Aug 2021)
TLS SNI support enabled
configure arguments: --prefix=/opt/homebrew/Cellar/openresty/1.19.9.1_2/nginx --with-cc-opt='-O2 -I/opt/homebrew/include -I/opt/homebrew/opt/pcre/include -I/opt/homebrew/opt/openresty-openssl111/include' --add-module=../ngx_devel_kit-0.3.1 --add-module=../echo-nginx-module-0.62 --add-module=../xss-nginx-module-0.06 --add-module=../ngx_coolkit-0.2 --add-module=../set-misc-nginx-module-0.32 --add-module=../form-input-nginx-module-0.12 --add-module=../encrypted-session-nginx-module-0.08 --add-module=../srcache-nginx-module-0.32 --add-module=../ngx_lua-0.10.20 --add-module=../ngx_lua_upstream-0.07 --add-module=../headers-more-nginx-module-0.33 --add-module=../array-var-nginx-module-0.05 --add-module=../memc-nginx-module-0.19 --add-module=../redis2-nginx-module-0.15 --add-module=../redis-nginx-module-0.3.7 --add-module=../ngx_stream_lua-0.0.10 --with-ld-opt='-Wl,-rpath,/opt/homebrew/Cellar/openresty/1.19.9.1_2/luajit/lib -L/opt/homebrew/lib -L/opt/homebrew/opt/pcre/lib -L/opt/homebrew/opt/openresty-openssl111/lib' --pid-path=/opt/homebrew/var/run/openresty.pid --lock-path=/opt/homebrew/var/run/openresty.lock --conf-path=/opt/homebrew/etc/openresty/nginx.conf --http-log-path=/opt/homebrew/var/log/nginx/access.log --error-log-path=/opt/homebrew/var/log/nginx/error.log --with-pcre-jit --with-ipv6 --with-stream --with-stream_ssl_module --with-stream_ssl_preread_module --with-http_v2_module --without-mail_pop3_module --without-mail_imap_module --without-mail_smtp_module --with-http_stub_status_module --with-http_realip_module --with-http_addition_module --with-http_auth_request_module --with-http_secure_link_module --with-http_random_index_module --with-http_geoip_module --with-http_gzip_static_module --with-http_sub_module --with-http_dav_module --with-http_flv_module --with-http_mp4_module --with-http_gunzip_module --with-threads --with-stream --with-http_ssl_module
```

- Debian 11的package_pach列表如下
```
/usr/local/openresty/nginx//lualib/?.lua;
/usr/local/openresty/site/lualib/?.ljbc;
/usr/local/openresty/site/lualib/?/init.ljbc;
/usr/local/openresty/lualib/?.ljbc;
/usr/local/openresty/lualib/?/init.ljbc;
/usr/local/openresty/site/lualib/?.lua;
/usr/local/openresty/site/lualib/?/init.lua;
/usr/local/openresty/lualib/?.lua;
/usr/local/openresty/lualib/?/init.lua;
./?.lua;
/usr/local/openresty/luajit/share/luajit-2.1.0-beta3/?.lua;
/usr/local/share/lua/5.1/?.lua;
/usr/local/share/lua/5.1/?/init.lua;
/usr/local/openresty/luajit/share/lua/5.1/?.lua;
/usr/local/openresty/luajit/share/lua/5.1/?/init.lua;
```

- Debian 11的configure参数
```
nginx version: openresty/1.19.9.1
built with OpenSSL 1.1.1l  24 Aug 2021
TLS SNI support enabled
configure arguments: --prefix=/usr/local/openresty/nginx --with-cc-opt='-O2 -DNGX_LUA_ABORT_AT_PANIC -I/usr/local/openresty/zlib/include -I/usr/local/openresty/pcre/include -I/usr/local/openresty/openssl111/include' --add-module=../ngx_devel_kit-0.3.1 --add-module=../echo-nginx-module-0.62 --add-module=../xss-nginx-module-0.06 --add-module=../ngx_coolkit-0.2 --add-module=../set-misc-nginx-module-0.32 --add-module=../form-input-nginx-module-0.12 --add-module=../encrypted-session-nginx-module-0.08 --add-module=../srcache-nginx-module-0.32 --add-module=../ngx_lua-0.10.20 --add-module=../ngx_lua_upstream-0.07 --add-module=../headers-more-nginx-module-0.33 --add-module=../array-var-nginx-module-0.05 --add-module=../memc-nginx-module-0.19 --add-module=../redis2-nginx-module-0.15 --add-module=../redis-nginx-module-0.3.7 --add-module=../ngx_stream_lua-0.0.10 --with-ld-opt='-Wl,-rpath,/usr/local/openresty/luajit/lib -L/usr/local/openresty/zlib/lib -L/usr/local/openresty/pcre/lib -L/usr/local/openresty/openssl111/lib -Wl,-rpath,/usr/local/openresty/zlib/lib:/usr/local/openresty/pcre/lib:/usr/local/openresty/openssl111/lib' --with-pcre-jit --with-stream --with-stream_ssl_module --with-stream_ssl_preread_module --with-http_v2_module --without-mail_pop3_module --without-mail_imap_module --without-mail_smtp_module --with-http_stub_status_module --with-http_realip_module --with-http_addition_module --with-http_auth_request_module --with-http_secure_link_module --with-http_random_index_module --with-http_gzip_static_module --with-http_sub_module --with-http_dav_module --with-http_flv_module --with-http_mp4_module --with-http_gunzip_module --with-threads --with-stream --with-http_ssl_module
```

而我的环境唯一区别就剩编译参数了。

附上我的Nginx编译参数
```
nginx version: openresty/1.19.9.1
built by gcc 8.5.0 20210514 (Red Hat 8.5.0-4) (GCC)
built with OpenSSL 1.1.1l  24 Aug 2021
TLS SNI support enabled
configure arguments: --with-cc-opt=-O2 --prefix=/app/nginx --with-http_stub_status_module --with-http_gzip_static_module --without-mail_pop3_module --without-mail_imap_module --without-mail_smtp_module --without-select_module --without-http_scgi_module --with-http_realip_module --with-http_auth_request_module --with-stream --with-stream_ssl_preread_module --with-stream_ssl_module --with-http_ssl_module --with-http_v2_module --with-http_sub_module --add-module=../ngx_devel_kit-0.3.1 --add-module=../ngx_lua-0.10.20 --add-module=../ngx_stream_lua-0.0.10 --add-module=../ngx_lua_upstream-0.07 --add-module=../set-misc-nginx-module-0.32 --add-module=../srcache-nginx-module-0.32 --add-module=../redis2-nginx-module-0.15 --add-module=../redis-nginx-module-0.3.7 --add-module=../echo-nginx-module-0.62 --add-module=../lua-var-nginx-module-0.5.2 --with-pcre=../pcre-8.44 --with-pcre-jit --with-openssl=../openssl-1.1.1l --with-openssl-opt=enable-tls1_3 --with-ld-opt=-Wl,-u,pcre_version,-rpath,/app/nginx/luajit/lib
```

## 解决方法(`推荐方法2`)

- 1、openssl暂放在默认搜索路径下，这样可以加载init.lua
```
/app/nginx/luajit/share/lua/5.1/resty/
```

- 2、查看lua-resty-openssl的测试样例的lua_package_path有单独配置init.lua，调整如下
```
lua_package_path "$prefix/lualib/?.lua;$prefix/lualib/?/init.lua;;";
```

调整后lua_package_path如下
```
/app/nginx//lualib/?.lua;
/app/nginx//lualib/?/init.lua;
./?.lua;
/app/nginx/luajit/share/luajit-2.1.0-beta3/?.lua;
/usr/local/share/lua/5.1/?.lua;
/usr/local/share/lua/5.1/?/init.lua;
/app/nginx/luajit/share/lua/5.1/?.lua;
/app/nginx/luajit/share/lua/5.1/?/init.lua;
```
