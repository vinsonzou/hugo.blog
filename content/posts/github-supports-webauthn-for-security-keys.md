+++
description = ""
date = "2019-08-23T15:05:22+08:00"
title = "GitHub supports WebAuthn"
tags = ["webauthn"]

+++

为了提高 GitHub 帐户的安全性，GitHub平台2019-08-21通过支持 Web 身份验证（WebAuthn）标准宣布了易于使用的身份验证选项。通过 WebAuthn，开发者们现在可以使用物理安全密钥在 GitHub 上进行双因素身份验证。如果你没有物理安全密钥，也可以将笔记本电脑或手机用作安全密钥。

以下是 GitHub 上支持的物理安全密钥的组合：

- Windows，macOS，Linux 和 Android：基于 Firefox 和 Chrome 的浏览器
- Windows：Edge
- macOS：Safari，目前在技术预览版中，但即将推出
- iOS：Brave 浏览器以及使用新的 YubiKey 5Ci

你还可以使用以下浏览器和生物识别选项：

- Windows 上的 Microsoft Edge，使用 Windows Hello（带面部识别，指纹识别器或 PIN）
- 在 macOS 的 Chrome 上使用 Touch ID
- 在 Android 的 Chrome 上使用指纹识别器

由于平台支持尚未普及，GitHub 目前支持安全密钥作为补充的第二个因素，将来，GitHub 会将安全密钥作为主要的第二因素。

来源：https://github.blog/2019-08-21-github-supports-webauthn-for-security-keys/
