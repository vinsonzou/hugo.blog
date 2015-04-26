+++
date = "2015-04-26T16:00:00+08:00"
draft = false
title = "利用TsunamiUDP加速跨机房迁移"
tags = ["TsunamiUDP"]
+++


## 部署

	yum -y install automake autoconf

	git clone git://github.com/rriley/tsunami-udp.git
	cd tsunami-udp
	./recompile.sh
	cp server/tsunamid client/tsunami /usr/bin

	#或者从sf.net下载
	wget http://iweb.dl.sourceforge.net/project/tsunami-udp/tsunami-udp/tsunami-v1.1-cvsbuild42/tsunami-v1.1-cvsbuild42.tar.gz
	tar zxf tsunami-v1.1-cvsbuild42.tar.gz
	cd tsunami-udp-v11-b42
	./recompile.sh
	cp server/tsunamid client/tsunami /usr/bin
	
## 使用

	#1、防火墙调整
	#服务端：开启TCP 46224(默认端口)
	#客户端：开启UDP 46224(默认端口)

	#2、开启服务端
	#待迁移文件都放在/app/game_data目录下(也可指定单文件传输)

	tsunamid --hbtimeout 60 /app/game_data/*

	#PS：这里设定心跳包超时时间为60秒，默认为15秒，在使用中很容易中断导致传输失败

	#3、开启客户端
	# 拉取服务端(122.225.100.100)的game_db.lz4文件，并限速100M(建议限制下，不然机房带宽就满了哦)
	tsunami set rate 100M connect 122.225.100.100 get gcmob_db.lz4

	# 拉取目录下所有文件
	tsunami set rate 100M connect 122.225.100.100 get \*

	#文档：http://tsunami-udp.cvs.sourceforge.net/viewvc/tsunami-udp/docs/USAGE.txt
	
	PS：未避免泄密，IP是随机填的
	
## 跨机房迁移示例

场景：
将14G文件从杭州机房迁移至北京机房

- 方法1：使用wget下载
![wget下载](http://m114-static.qiniudn.com/img/wget.png)

- 方法2：使用TsunamiUDP工具
![TsunamiUDP下载](http://m114-static.qiniudn.com/img/TsunamiUDP.jpg)

总结:

TsunamiUDP相比wget优势太明显了，TsunamiUDP把带宽能跑满，而wget的速度不敢恭维。

## 参考

http://sysadminandnetworking.blogspot.com/2013/06/tsunami-udp-faster-than-rsync.html
http://blog.csdn.net/awschina/article/details/38661889