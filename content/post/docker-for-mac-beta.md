+++
date = "2016-05-20T00:05:35+08:00"
tags = ["Docker"]
title = "Docker for Mac Beta尝鲜"
topics = ["Docker"]

+++

## Docker for Mac

Docker for Mac 是一个原生的苹果应用程序，被安装到 `/Application` 目录。安装时会创建 `/usr/local/bin` 目录下的 `docker`、`docker-compose`、`docker-machine` 符号链接，这些符号链接指向 `~/Library/Group Containers/group.com.docker/bin` 目录下的各类文件，而 `~/Library/Group Containers/group.com.docker/bin` 的文件实际上也是符号链接，他们指向 `/Applications/Docker.app/Contents/Resources/bin` 目录下的实际二进制文件。

* Docker for Mac 使用通过 Hypervisor.framework 提供的轻量级的 [xhyve](https://github.com/mist64/xhyve) 虚拟化技术
* Docker for Mac 不使用 docker-machine 管理虚拟机
* Docker for Mac 不通过 TCP 端口通信，反而使用 /var/tmp/docker.sock 套接字文件通信（实际上是将 /var/tmp 目录挂载到了虚拟机中，虚拟机在其中生成套接字文件）
* 由于使用了 xhyve 虚拟机，所以可以模拟不同架构的处理器，这样开发者就直接能在 Mac 上使用 Docker 使用诸多平台的镜像文件，比如 arm 等。

为了能主机虚拟机共享文件，Docker 使用 osxfs 作为全新的文件共享方案，在很多方面都有全新的特性，比如在文件权限、命名空间、文件所有者、文件系统事件、挂载点、符号链接、文件类型、扩展属性等方面都有了全新的内容，并且，所有产生的日志都能通过 syslog 查询，非常方便。不过现在依旧存在许多问题，比如没有设置 docker daemon 各项参数的接口。

## 如何为Docker Engine设置代理

```
screen ~/Library/Containers/com.docker.docker/Data/com.docker.driver.amd64-linux/tty
```

敲一下回车，登录，用户名root，没有密码，直接回车。编辑/etc/init.d/docker文件，如下添加代理：

```
start-stop-daemon --start --quiet \
-e HTTP_PROXY=http(s)://proxy_host:proxy_port \
--background \
--exec ${command} \
--pidfile ${pidfile} \
--stderr "${DOCKER_LOGFILE}" \
--stdout "${DOCKER_LOGFILE}" \
-- daemon --pidfile=${pidfile} ${DOCKER_OPTS}
```

重启Docker服务
```
/etc/init.d/docker restart
/etc/init.d/docker status
```

## 如何添加registry-mirror

由于你懂的原因，国内拉镜像非常慢，甚至根本无法下载，解决方法如下：

导出:
```
pinata get daemon > myconfig.json
```

将myconfig.json修改为:
```
{"storage-driver":"aufs","debug":true,"registry-mirrors":["https://docker.mirrors.ustc.edu.cn"]}
```

导入:
```
pinata set daemon @myconfig.json
```

重新启动Docker服务即可生效！
