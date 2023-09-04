---
title: "Debian 12配置静态IP"
subtitle: ""
date: 2023-09-04T23:00:00+08:00
lastmod: 2023-09-04T23:0:00+08:00
description: ""

tags: ["SRE"]
categories: ["SRE"]
---

最近体验Debian 12，发现静态IP配置方式与之前版本不太一样了，下面我们就来探究探究。

## Netplan介绍

在默认情况下，Debian 12 使用 Netplan 来配置网络。Netplan 是一个网络配置工具，它为 systemd-networkd 提供了配置文件。

在 Debian 12 中，Netplan 配置文件位于 `/etc/netplan/ `目录中，以 `.yaml` 扩展名结尾。默认情况下，Netplan配置文件会转换为相应的 systemd-networkd 配置文件，并存储在 `/run/systemd/network/` 目录中。

> `/run/systemd/network/` 目录是一个临时目录，其中的配置文件不会永久保存。每次系统启动时，Netplan 配置文件都会被读取并转换为相应的 systemd-networkd 配置文件，然后加载到 systemd-networkd 中使用。

默认情况下，系统会读取和使用 `/run/systemd/network/` 目录下的 `/run/systemd/network/10-netplan-all-en.network`文件作为 systemd-networkd 的配置文件来配置网络接口。

默认配置文件示例：`/etc/netplan/90-default.yaml`

```yaml
network:
    version: 2
    ethernets:
        all-en:
            match:
                name: en*
            dhcp4: true
            dhcp4-overrides:
                use-domains: true
            dhcp6: true
            dhcp6-overrides:
                use-domains: true
        all-eth:
            match:
                name: eth*
            dhcp4: true
            dhcp4-overrides:
                use-domains: true
            dhcp6: true
            dhcp6-overrides:
                use-domains: true
```

## 使用netplan配置静态IP

`vim /etc/netplan/01-netcfg.yaml`

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    enp1s0:
      addresses: [192.168.1.100/24]
      gateway4: 192.168.1.1
      nameservers:
        addresses: [192.168.1.1]
```

使用以下命令应用新的 Netplan 配置

```bash
mv /etc/netplan/90-default.yaml{,.bak}
sudo netplan apply
```

这将重新加载网络配置并使更改生效。

确认网络配置是否生效：

```bash
ip addr show enp1s0
```

### 配置优先级

- 文件名称：在 `/etc/netplan/` 目录中，Netplan 将按照文件名的字母顺序依次处理配置文件。较小的数字或字母表示更早处理，较大的数字或字母表示更晚处理。

  例如，如果存在两个配置文件 `01-netcfg.yaml` 和 `02-custom.yaml`，那么 `01-netcfg.yaml` 将在 `02-custom.yaml` 之前被处理。

- 文件内容：每个配置文件中定义的网络接口块也有优先级。较早出现的接口块将具有较高的优先级。

  ```yaml
  network:
    version: 2
    renderer: networkd
    ethernets:
      eth0:  # 较早出现的接口块
        addresses: [192.168.1.100/24]
      eth1:  # 较晚出现的接口块
        addresses: [192.168.2.100/24]
  ```

