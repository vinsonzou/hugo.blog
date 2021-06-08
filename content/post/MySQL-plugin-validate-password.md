+++
description = ""
date = "2021-06-08T13:30:08+08:00"
title = "MySQL插件之密码复杂性"
tags = ["MySQL", "等保"]
topics = ["MySQL"]

+++

## 目录

- [等保需求](#等保需求)
- [插件介绍](#插件介绍)
- [插件安装](#插件安装)
- [插件配置](#插件配置)


## 等保需求

- 用户密码复杂度：3种字符以上，至少8位长度。使用 validate_password 插件实现。

## 插件介绍

MySQL 5.6/5.7 上密码复杂度策略实现, MySQL 8.0有组件实现方式，同时也支持插件方式。

## 插件安装

```sh
-- 配置文件增加以下配置
[mysqld]
plugin-load-add      = validate_password.so
validate-password    = FORCE_PLUS_PERMANENT

-- 插件动态安装启用
mysql> INSTALL PLUGIN validate_password soname 'validate_password.so';

-- 验证是否正常安装
mysql> SELECT PLUGIN_NAME, PLUGIN_LIBRARY, PLUGIN_STATUS, LOAD_OPTION 
FROM INFORMATION_SCHEMA.PLUGINS 
WHERE PLUGIN_NAME = 'validate_password';

mysql> SHOW PLUGINS;

注意：参数FORCE_PLUS_PERMANENT是为了防止插件在MySQL运行的时候被卸载，如下所示，当你卸载插件时就会报错：
mysql> UNINSTALL PLUGIN validate_password;
ERROR 1702 (HY000): Plugin 'validate_password' is force_plus_permanent and can not be unloaded
```

## 插件配置

```sh
-- 查看默认相关变量
mysql> show variables like 'validate_password%';
+--------------------------------------+--------+
| Variable_name                        | Value  |
+--------------------------------------+--------+
| validate_password_check_user_name    | OFF    |
| validate_password_dictionary_file    |        |
| validate_password_length             | 8      |
| validate_password_mixed_case_count   | 1      |
| validate_password_number_count       | 1      |
| validate_password_policy             | MEDIUM |
| validate_password_special_char_count | 1      |
+--------------------------------------+--------+

mysql> select password('abc');
ERROR 1819 (HY000): Your password does not satisfy the current policy requirements
mysql> select password('abc123');
ERROR 1819 (HY000): Your password does not satisfy the current policy requirements
mysql> select password('abc123ABC');
ERROR 1819 (HY000): Your password does not satisfy the current policy requirements
mysql> select password('abc123@A');

所以你更改密码必须满足：数字、小写字母、大写字母 、特殊字符、长度至少8位
```

参数说明:

- validate-password=ON/OFF/FORCE/FORCE_PLUS_PERMANENT: 决定是否使用该插件(及强制/永久强制使用)。
- validate_password_dictionary_file：插件用于验证密码强度的字典文件路径。
- validate_password_length：密码最小长度。
- validate_password_mixed_case_count：密码至少要包含的小写字母个数和大写字母个数。
- validate_password_number_count：密码至少要包含的数字个数。
- validate_password_policy：密码强度检查等级，0/LOW、1/MEDIUM、2/STRONG。
- validate_password_special_char_count：密码至少要包含的特殊字符数。

关于validate_password_policy-密码强度检查等级：

```
Policy	        Tests Performed
0 or LOW   	Length
1 or MEDIUM	Length; numeric, lowercase/uppercase, and special characters
2 or STRONG	Length; numeric, lowercase/uppercase, and special characters; dictionary file
```
