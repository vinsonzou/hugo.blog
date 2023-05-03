---
title: "解决Android安装包签名异常"
subtitle: ""
date: 2023-04-22T16:03:03+08:00
lastmod: 2023-04-22T16:03:03+08:00
description: ""

tags: ["Android"]
categories: ["Android"]
---

## 环境

- Unity 2021.3
  - Android Gradle Plugin(AGP): 4.0.1
  - Gradle: 6.1.1
- APK 大小: 2.1G
- 系统版本: Android 12

使用`adb`安装报错如下

```bash
adb install /tmp/com.test.demo_0.0.1_18971.apk
/tmp/com.test.demo_0.0.1_18971.apk
Performing Streamed Install
adb: failed to install /tmp/com.test.demo_0.0.1_18971.apk: Failure [INSTALL_PARSE_FAILED_NO_CERTIFICATES: Failed collecting certificates for /data/app/vmdl226784575.tmp/base.apk: Failed to collect certificates from /data/app/vmdl226784575.tmp/base.apk using APK Signature Scheme v2: integer overflow]
```

之前 APK 安装一直正常，最近适配渠道商 SDK，调整了 AGP 和 Gradle 版本后，就出现了报错。

针对环境变化部分，使用不同 AGP 版本和 Gradle 版本进行验证，验证结果如下
|出包类型|AGP|Gradle|BuildTools|出包结果|Android 12 安装|备注|
|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
|release|4.0.1|6.1.1|29.0.2|✅|❌||
|release|4.0.1|6.5.1|29.0.2|✅|❌||
|release|4.0.1|7.5.1|29.0.2|❌|❌||
|release|4.1.3|6.5.1|29.0.2|✅|✅||
|release|4.1.3|6.7.1|29.0.2|✅|✅||
|release|4.1.3|7.5.1|29.0.2|✅|✅||
|release|4.2.2|7.5.1|29.0.2|✅|✅|未对接 SDK 前版本|

通过验证数据，目前针对这个问题，先升级`AGP 4.1.3`解决问题。

## 补充

网上各种查询，也有说是安装包超过 2G 导致的，我去华为应用商店去看各大游戏安装包大小，发现个现象就是绝大部分游戏安装包都严格控制在 2G 以内。如相关组件无法升级，就只能压缩安装包大小去解决了。

## 参考

- [Unity2021.3 Gradle 版本约束](https://docs.unity3d.com/Manual/android-gradle-overview.html)
- [Android Gradle 插件版本说明](https://developer.android.com/studio/releases/gradle-plugin?hl=zh-cn)
- https://lemmasoft.renai.us/forums/viewtopic.php?t=63244
- https://forum.unity.com/threads/using-apk-signature-scheme-v2-integer-overflow-when-installing-apk-to-quest.1204852/
- https://github.com/Meituan-Dianping/walle/issues/256