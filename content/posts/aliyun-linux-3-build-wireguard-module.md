---
title: "Alibaba Cloud Linux 3编译wireguard模块"
subtitle: ""
date: 2021-10-22T16:01:09+08:00
lastmod: 2021-10-22T16:01:09+08:00
description: ""

tags: ["linux"]
categories: []
---

因wireguard已经合并至5.6内核中，刚好Alibaba Cloud Linux 3是5.10内核，结果测试下来没有wireguard模块，
因是CentOS 8的定制版又用不了CentOS 8相关模块，当前就只能自己动手编译内核模块了。这次第一次编译内核模块，
如有错误还望指正。

## 1. 环境

- OS: Alibaba Cloud Linux 3
- 镜像版本: `aliyun_3_x64_20G_alibase_20210910.vhd`
- 内核版本: `5.10.60-9.al8.x86_64`

## 2. 编译准备

```
dnf -y install flex bison kernel-devel-`uname -r` elfutils-libelf-devel openssl-devel
```

通过 `make menuconfig` 找出 wireguard 依赖模块与当前/boot/config-\`uname -r\`差异
```
Device Drivers  --->
    [*] Network device support  --->
        [*] Network core driver support
        <*>   WireGuard secure network tunnel
```

## 3. 编译模块

```
wget https://cdn.kernel.org/pub/linux/kernel/v5.x/linux-5.10.60.tar.xz
tar Jxf linux-5.10.60.tar.xz
cd linux-5.10.60
cp /boot/config-`uname -r` ./.config

make LOCALVERSION=".al8.x86_64" modules_prepare
make M=drivers/net/wireguard CONFIG_WIREGUARD=m modules
make M=lib/crypto CONFIG_CRYPTO_LIB_CURVE25519_GENERIC=m CONFIG_CRYPTO_LIB_CURVE25519=m \
                  CONFIG_CRYPTO_LIB_BLAKE2S=m CONFIG_CRYPTO_LIB_BLAKE2S_GENERIC=m \
                  CONFIG_CRYPTO_LIB_CHACHA20POLY1305=m modules
```

### 3.1 编译模块放到指定位置

```
mkdir /lib/modules/`uname -r`/kernel/drivers/net/wireguard
mv drivers/net/wireguard/wireguard.ko /lib/modules/`uname -r`/kernel/drivers/net/wireguard/

mv lib/crypto/libblake2s*.ko /lib/modules/`uname -r`/kernel/lib/crypto/
mv lib/crypto/libcurve25519*.ko /lib/modules/`uname -r`/kernel/lib/crypto/
mv lib/crypto/libchacha20poly1305.ko /lib/modules/`uname -r`/kernel/lib/crypto/
```

### 3.2 建立依赖

```
cd /lib/modules/`uname -r`/
depmod -a
```

### 3.3 加载模块

```
modprobe wireguard
```

### 3.4 查看模块依赖

```
[root@localhost 5.10.60-9.al8.x86_64]# lsmod |grep wireguard
lsmod
wireguard              90112  0
libblake2s             16384  1 wireguard
libchacha20poly1305    16384  1 wireguard
libcurve25519_generic    49152  1 wireguard
ip6_udp_tunnel         16384  1 wireguard
udp_tunnel             28672  1 wireguard
```

具体依赖库查看
```
[root@localhost 5.10.60-9.al8.x86_64]# strace modprobe wireguard

...其他strace项忽略
# 当前编译的模块
/lib/modules/5.10.60-9.al8.x86_64/kernel/lib/crypto/libchacha20poly1305.ko
/lib/modules/5.10.60-9.al8.x86_64/kernel/lib/crypto/libcurve25519-generic.ko
/lib/modules/5.10.60-9.al8.x86_64/kernel/lib/crypto/libblake2s.ko
/lib/modules/5.10.60-9.al8.x86_64/kernel/lib/crypto/libblake2s-generic.ko

# 内置模块
/lib/modules/5.10.60-9.al8.x86_64/kernel/arch/x86/crypto/chacha-x86_64.ko
/lib/modules/5.10.60-9.al8.x86_64/kernel/arch/x86/crypto/poly1305-x86_64.ko
/lib/modules/5.10.60-9.al8.x86_64/kernel/net/ipv6/ip6_udp_tunnel.ko
/lib/modules/5.10.60-9.al8.x86_64/kernel/net/ipv4/udp_tunnel.ko
/lib/modules/5.10.60-9.al8.x86_64/kernel/lib/crypto/libchacha.ko
/lib/modules/5.10.60-9.al8.x86_64/kernel/net/ipv4/udp_tunnel.ko
/lib/modules/5.10.60-9.al8.x86_64/kernel/net/ipv6/ip6_udp_tunnel.ko
...
```

## 4. FAQ

**1. 仅编译wireguard模块报错如下**

```
[Fri Oct 22 13:57:48 2021] wireguard: no symbol version for module_layout
[Fri Oct 22 13:57:48 2021] wireguard: loading out-of-tree module taints kernel.
[Fri Oct 22 13:57:48 2021] wireguard: Unknown symbol blake2s_final (err -2)
[Fri Oct 22 13:57:48 2021] wireguard: Unknown symbol curve25519_null_point (err -2)
[Fri Oct 22 13:57:48 2021] wireguard: Unknown symbol chacha20poly1305_encrypt_sg_inplace (err -2)
[Fri Oct 22 13:57:48 2021] wireguard: Unknown symbol blake2s_update (err -2)
[Fri Oct 22 13:57:48 2021] wireguard: Unknown symbol chacha20poly1305_encrypt (err -2)
[Fri Oct 22 13:57:48 2021] wireguard: Unknown symbol chacha20poly1305_decrypt_sg_inplace (err -2)
[Fri Oct 22 13:57:48 2021] wireguard: Unknown symbol xchacha20poly1305_encrypt (err -2)
[Fri Oct 22 13:57:48 2021] wireguard: Unknown symbol xchacha20poly1305_decrypt (err -2)
[Fri Oct 22 13:57:48 2021] wireguard: Unknown symbol curve25519_base_point (err -2)
[Fri Oct 22 13:57:48 2021] wireguard: Unknown symbol chacha20poly1305_decrypt (err -2)
[Fri Oct 22 13:57:48 2021] wireguard: Unknown symbol blake2s256_hmac (err -2)
[Fri Oct 22 13:57:48 2021] wireguard: Unknown symbol curve25519_generic (err -2)
```

添加如下依赖可修复:
- `CONFIG_CRYPTO_LIB_CURVE25519`
- `CONFIG_CRYPTO_LIB_CURVE25519_GENERIC`
- `CONFIG_CRYPTO_LIB_CHACHA20POLY1305`
- `CONFIG_CRYPTO_LIB_BLAKE2S`

**2. 添加CONFIG_CRYPTO_LIB_BLAKE2S_GENERIC=m修复此报错**

```
[Fri Oct 22 14:58:24 2021] libblake2s: Unknown symbol blake2s_compress_generic (err -2)
```
