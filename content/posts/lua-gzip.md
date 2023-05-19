---
title: "Lua实现返回内容gzip解压/压缩"
subtitle: ""
date: 2023-05-12T12:43:03+08:00
lastmod: 2023-05-12T12:43:03+08:00
description: ""

tags: ["OpenResty"]
categories: ["OpenResty"]
---

## 背景

最近要获取 jira 返回数据更新关注列表，jira 服务器的请求和返回的数据默认进行了 gzip 压缩。目前有 2 种方式

- jira 设置中关闭 gzip 压缩
- 使用 lua 对 gzip 进行解压，参考库[lua-zlib](https://github.com/brimworks/lua-zlib)

## HTTP 压缩过程

可能有的小伙伴，不知道传输内容 gzip 压缩是什么？是一个什么样的过程，在这里科普一下。

在 HTTP 协议中，可以对内容（也就是 body 部分）进行编码，可以采用 gzip 这样的编码，达到压缩的目的。也可以使用其它的编码对内容搅乱或加密，以此来防止未授权的第三方看到传输的内容。一般 HTTP 比较通用的压缩算法就是 gzip，通过 gzip 来压缩 html、javascript、CSS 文件。能大大减少网络传输的数据量，提高用户显示网页的熟读，当然同时会增加一点点服务器的开销。

下面简单说下压缩的过程：

- 浏览器发送 Http Request 给 Web 服务器, request 中有 Accept-Encoding: gzip,deflate。 (告诉服务器， 浏览器支持 gzip 压缩)
- Web 服务器接到 Request 后， 生成原始的 Response, 其中有原始的 Content-Type 和 Content-Length
- Web 服务器通过 Gzip，来对 Response 进行编码，编码后 header 中有 Content-Type 和 Content-Length (压缩后的大小)，并且增加了 Content-Encoding:gzip。然后把 Response 发送给浏览器
- 浏览器接到 Response 后，根据 Content-Encoding:gzip 来对 Response 进行解码。获取到原始 Response 后，然后显示出网页

## Lua-zlib

使用源码编译步骤如下

```bash
wget https://github.com/brimworks/lua-zlib/archive/master.zip
unzip master.zip
cd lua-zlib-master
cmake -DLUA_INCLUDE_DIR=/usr/local/openresty/luajit/include/luajit-2.1 -DLUA_LIBRARIES=/usr/local/openresty/luajit/lib -DUSE_LUAJIT=ON -DUSE_LUA=OFF
make && make install
```

编译后，将编译生成的 `zlib.so` 复制到 `/usr/local/openresty/lualib/` 目录下

首先使用 cmake 来生成编译配置文件

- `LUA_INCLUDE_DIR` 指定 luajit 的 include 文件夹
- `LUA_LIBRARIES` 指定 luajit 的 lib 文件夹
- `USE_LUAJIT = ON` 和 `USE_LUA = OFF` 指定我们使用的是 luajit

使用方式

```lua
local zlib = require("zlib")

-- 压缩
local compress = zlib.deflate()(body, "finish")

-- 解压缩
local uncompress = zlib.inflate()(compress)
```

使用起来是不是很简单，我以为就要大功告成了，结果出现了问题，原本这个插件的作用就是将服务器返回的压缩过的数据进行解压并解析，过滤一些敏感的信息，再压缩继续返回，解压解析一切正常，再压缩就出了问题，原来是因为 gzip 有附加的 header，而如果在使用 deflate() 方法不设置参数的话，默认得到的是 zlib 压缩内容，而不是 gzip 压缩内容，因此压缩的使用方式需要修改下：

```lua
local zlib = require("zlib")

-- input:  string
-- output: string compressed with gzip
function compress(str)
   local level = 5
   local windowSize = 15+16
   return zlib.deflate(level, windowSize)(str, "finish")
end
```

在文档里有说明 function stream = zlib.deflate([ int compression_level ], [ int window_size ])

```
If no compression_level is provided uses Z_DEFAULT_COMPRESSION (6),
compression level is a number from 1-9 where zlib.BEST_SPEED is 1
and zlib.BEST_COMPRESSION is 9.

Returns a "stream" function that compresses (or deflates) all
strings passed in.  Specifically, use it as such:

string deflated, bool eof, int bytes_in, int bytes_out =
        stream(string input [, 'sync' | 'full' | 'finish'])

    Takes input and deflates and returns a portion of it,
    optionally forcing a flush.

    A 'sync' flush will force all pending output to be flushed to
    the return value and the output is aligned on a byte boundary,
    so that the decompressor can get all input data available so
    far.  Flushing may degrade compression for some compression
    algorithms and so it should be used only when necessary.

    A 'full' flush will flush all output as with 'sync', and the
    compression state is reset so that decompression can restart
    from this point if previous compressed data has been damaged
    or if random access is desired. Using Z_FULL_FLUSH too often
    can seriously degrade the compression.

    A 'finish' flush will force all pending output to be processed
    and results in the stream become unusable.  Any future
    attempts to print anything other than the empty string will
    result in an error that begins with IllegalState.

    The eof result is true if 'finish' was specified, otherwise
    it is false.

    The bytes_in is how many bytes of input have been passed to
    stream, and bytes_out is the number of bytes returned in
    deflated string chunks.
```

搞定收工！