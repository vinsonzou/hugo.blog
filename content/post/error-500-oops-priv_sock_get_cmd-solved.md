+++
tags = ["vsftpd"]
date = "2017-07-23T16:21:45+08:00"
title = "Error: 500 OOPS: priv_sock_get_cmd [SOLVED]"
description = ""
topics = ["Troubleshooting"]

+++

## Troubleshooting

**seccomp filter sanboxing with vsftpd 3.0.x**

The following error may occur on ftp clients with vsftpd 3.0.x:

```sh
500 OOPS: priv_sock_get_cmd
```

This is caused by [seccomp filter sanboxing](http://en.wikipedia.org/wiki/Seccomp), which is enabled by default on `amd64`. To workaround this issue, disable seccomp filter sanboxing:

```sh
echo 'seccomp_sandbox=NO' >> vsftpd.conf
service vsftpd restart
```

For further information, refer to [Red Hat bug #845980](https://bugzilla.redhat.com/show_bug.cgi?id=845980).
