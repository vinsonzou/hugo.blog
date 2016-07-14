+++
date = "2016-07-12T23:29:35+08:00"
description = ""
tags = ["idrac"]
title = "R420服务器idrac连接用户数超限制"
topics = []

+++

Dell R420服务器中，比较经常出现idrac无法连接，或者连接用户数超限(`RAC0218错误`)的问题，升级iDrac卡firmware至1.57.57即可解决。

该版本的bug fix中提到过一点：

- Fix for issues that cause iDRAC7 sluggish responsiveness after a prolonged period of time (approx. 45-100 days, depending on the usage). In some cases, if the iDRAC is not reset, the iDRAC may become unresponsive and requires a server AC Power on reset. This issue was introduced in firmware release 1.50.50 and fixed in 1.56.55.

*RAC0218: The maximum number of user sessions is reached.*
![](http://m114-static.qiniudn.com/img/Dell_iDRAC_7_Enterprise_RAC0218_-_The_maximum_number_of_user_sessions_is_reached.jpg)

**2种临时解决方法(idrac重新初始化)**

* 长按前面板i键20秒，idrac会重启初始化  （建议使用这种方法，无需重启服务器）
* 关机后拔掉电源线，长按电源开关按键15秒，再插上电源线开机

**tftp server准备**

* [1.57.57](http://www.dell.com/support/home/us/en/04/Drivers/DriversDetails?driverId=XH6FX)
* [1.66.65](http://www.dell.com/support/home/us/en/04/Drivers/DriversDetails?driverId=3F4WV)
* [当前最新版2.30.30.30](http://www.dell.com/support/home/us/en/19/Drivers/DriversDetails?driverId=JHF76)
    * 支持HTML5 virtual console and virtual media

```
#升级以1.57.57为例，也可选择更高版本使用
wget http://downloads.dell.com/FOLDER02171695M/1/ESM_Firmware_XH6FX_LN_1.57.57_A00.BIN
sh ESM_Firmware_XH6FX_LN_1.57.57_A00.BIN --extract ./idrac7-1.57.57
cp ./idrac7-1.57.57/payload/firmimg.d7 /var/lib/tftpboot/dell/idrac7-1.57.57
```

**firmware升级方法**

* [通过iDrac7 Web GUI升级](http://en.community.dell.com/techcenter/b/techcenter/archive/2013/04/17/idrac7-now-supports-updating-server-components-using-racadm-and-web-gui)
* 通过ssh登录升级

```
#ssh登录后进行firmware更新
ssh root@iDrac_IP
racadm fwupdate -g -u -a $tftp_server_ip -d /dell/idrac7-1.57.57
```

* 通过racadm命令行升级

```
racadm -r iDrac_IP -u root -p calvin fwupdate -g -u -a $tftp_server_ip -d /dell/idrac7-1.57.57
```
