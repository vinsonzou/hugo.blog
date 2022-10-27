---
title: "fastlane无法上传ipa至TestFlight"
subtitle: ""
date: 2022-10-22T15:08:13+08:00
lastmod: 2022-10-22T15:08:13+08:00
description: ""

tags: ["fastlane"]
categories: ["fastlane"]
---

## 环境

- Mac Studio（2022）
  - MacOS 12.5.1
    - XCode 13.4.1
    - fastlane 2.210.1
    - ruby 3.1.2p20 (2022-04-12 revision 4491bb740a) [arm64-darwin21]

## 问题

上周还能正常上传TestFlight，今天就出现这诡异的问题，报错日志如下

```
[14:08:17]: [iTMSTransporter] [2022-10-22 14:08:16 CST] <main> DBG-X:   parameter ErrorMessage = An error occurred while trying to call the requested method validateAssets. (1272)

[14:08:17]: [iTMSTransporter] [2022-10-22 14:08:16 CST] <main> DBG-X:   parameter ShouldUseRESTAPIs = false

[14:08:17]: [iTMSTransporter] [2022-10-22 14:08:16 CST] <main> DBG-X:   parameter Success = false

[14:08:17]: [iTMSTransporter] [2022-10-22 14:08:16 CST] <main> ERROR: An error occurred while trying to call the requested method validateAssets. (1272)

[14:08:17]: [iTMSTransporter] [2022-10-22 14:08:16 CST] <main> DBG-X: The error code is: 1272

[14:08:17]: [iTMSTransporter] [2022-10-22 14:08:16 CST] <main>  INFO: Done performing authentication.

[14:08:17]: [iTMSTransporter]

[14:08:17]: [iTMSTransporter]

[14:08:17]: [iTMSTransporter]

[14:08:17]: [iTMSTransporter] Package Summary:

[14:08:17]: [iTMSTransporter]

[14:08:17]: [iTMSTransporter] 1 package(s) were not uploaded because they had problems:

[14:08:17]: [iTMSTransporter] 	/var/folders/jl/qqfvhns973q9tx0tvtgv7sdh0000gn/T/d20221022-76139-wbw4gm/1636622554-49ca9117-4e07-4848-b9d4-d4a305b1ffa2.itmsp - Error Messages:

[14:08:17]: [iTMSTransporter] 		An error occurred while trying to call the requested method validateAssets. (1272)

[14:08:17]: [iTMSTransporter] [2022-10-22 14:08:16 CST] <main> DBG-X: Returning 1
```

## 解决方法

在fastlane github上一堆讨论，各种方法都尝试了一次，终于能上传ipa至TestFlight了。几个点说下

- 根源是自动升级了iTMSTransporter，从2.3.0升级至3.0.0
- fastlane 使用altool替代iTMSTransporter，altool是XCode 14默认选项，[PR链接](https://github.com/fastlane/fastlane/pull/20631)

验证

- 在Xcode 13下，手动命令行试试altool能否正常验证和上传ipa

```
# 校验APP
API_PRIVATE_KEYS_DIR=fastlane xcrun altool --validate-app -t ios --apiKey <KEY_ID> --apiIssuer <ISSUER_ID> -f <PATH_TO_IPA>.ipa

# 上传IPA至TestFlight
API_PRIVATE_KEYS_DIR=fastlane xcrun altool --upload-app -t ios --apiKey <KEY_ID> --apiIssuer <ISSUER_ID> -f <PATH_TO_IPA>.ipa
```

结果显示使用altool能正常上传。

使用altool就能解决无法上传的问题。

## 总结

- Xcode版本可以升级，直接升级至Xcode 14；
- 如Xcode版本无法升级，则手动修改fastlane对xcode版本的限制；

## 参考

- [Error while trying to upload the ipa to testflight #20636](https://github.com/fastlane/fastlane/discussions/20636)
- [[pilot] fails to upload build to TestFlight using api key after iTMSTransporter auto updated to version 3.0.0 with An exception has occurred: issuerId is required error #20741](https://github.com/fastlane/fastlane/issues/20741)
- [bug(ios): transporter 3.0.0 crashes during upload to App Store #7483](https://github.com/keymanapp/keyman/issues/7483)
- https://stackoverflow.com/questions/72006870/itmstransporter-could-not-generate-an-itmsp-null
