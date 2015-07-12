+++
date = "2015-04-25T17:16:29+08:00"
menu = "main"
tags = ["Zabbix", "Telegram"]
title = "Zabbix使用Telegram发送报警"

+++

### 环境

CentOS 6.X x86_64

### 编译Telegram

    yum -y install lua-devel openssl-devel libconfig-devel readline-devel libevent-devel

    git clone --recursive https://github.com/vysheng/tg.git
    cd tg
    ./configure
    make

    mkdir /usr/local/tg
    cp tg-server.pub /usr/local/tg
    cp bin/telegram-cli /usr/local/tg


zabbix报警脚本/usr/local/tg/telegram.sh:

    #!/bin/sh

    cd `dirname $0`
    ./telegram-cli -k tg-server.pub -WDCRE -P 8890 -d &>/dev/null &

### Zabbix报警配置

将如下Zabbix Server配置注释并修改如下
AlertScriptsPath=/usr/local/zabbix/alertscripts

`/usr/local/zabbix/alertscripts/tg.sh`内容如下:

    #!/bin/sh

    export to=$1;
    export subject=$2;
    export body=$3;

    echo -e "msg $to ${subject}_#_${body}" | nc localhost 8890
    #注意事项: body只能有一行内容，超过一行的内容是不会发送的。

Zabbix添加Media types
![media_Telegram](http://m114-static.qiniudn.com/img/media_Telegram.png)

PS：Telegram已于2015年7月10日被天朝和谐。。。