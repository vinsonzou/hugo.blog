+++
description = ""
date = "2021-05-18T10:08:00+08:00"
title = "golang encrypt/decrypt by MacOS keychain"
tags = ["golang"]

+++

功能如下:

- AES加密/解密
- AES秘钥从MacOS keychain读取
- 支持Data At Rest Encryption (DARE)，加密文件内存占用小于100K，参考 [sio](https://github.com/minio/sio)

{{< gist vinsonzou 3576a4d20d3247ec0a1953d488e7d446 >}}
