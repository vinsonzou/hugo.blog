---
title: "Tomcat 7.0.76 Invalid character found in the request target"
subtitle: ""
date: 2017-03-27T13:08:25+08:00
lastmod: 2017-03-27T13:08:25+08:00
description: ""

tags: ["tomcat"]
categories: ["Troubleshooting"]
---

**故障现象**

升级tomcat至7.0.76后，GET请求的参数中含有中文时tomcat返回400错误，tomcat错误日志如下

```java
java.lang.IllegalArgumentException: Invalid character found in the request target. The valid characters are defined in RFC 7230 and RFC 3986
	at org.apache.coyote.http11.InternalNioInputBuffer.parseRequestLine(InternalNioInputBuffer.java:317)
	at org.apache.coyote.http11.AbstractHttp11Processor.process(AbstractHttp11Processor.java:1000)
	at org.apache.coyote.AbstractProtocol$AbstractConnectionHandler.process(AbstractProtocol.java:637)
	at org.apache.tomcat.util.net.NioEndpoint$SocketProcessor.doRun(NioEndpoint.java:1756)
	at org.apache.tomcat.util.net.NioEndpoint$SocketProcessor.run(NioEndpoint.java:1715)
	at java.util.concurrent.ThreadPoolExecutor$Worker.runTask(ThreadPoolExecutor.java:886)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:908)
	at org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61)
	at java.lang.Thread.run(Thread.java:662)
```

**原因**

  查询[Changelog](http://tomcat.apache.org/tomcat-7.0-doc/changelog.html#Tomcat_7.0.73_(violetagg))得知，tomcat 7.0.73版本添加了**Add additional checks for valid characters to the HTTP request line parsing so invalid request lines are rejected sooner**导致。

**解决方法**

* 请求前自行转义
* 更换tomcat为较低版本(不过tomcat的这次更改是依据RFC7230 and RFC 3986,在往后的版本,不会移除该特性)
