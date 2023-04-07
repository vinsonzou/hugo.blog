---
title: "git clone Out of memory, malloc failed"
subtitle: ""
date: 2023-04-07T14:40:25+08:00
lastmod: 2023-04-07T14:40:25+08:00
description: ""

tags: ["git"]
categories: ["Troubleshooting"]
---

**报错如下**
```
[root@localhost ~]# git clone xxx/iOSClient.git
Cloning into 'iOSClient'...
remote: Enumerating objects: 4455, done.
remote: Counting objects: 100% (4455/4455), done.
remote: warning: suboptimal pack - out of memory
remote: error: Out of memory, malloc failed (tried to allocate 343057209 bytes)
remote: fatal: packed object 98d3df78eb15e9010125e8d6c005f2e0a3792164 (stored in ./objects/pack/pack-c077e52335fd57f98e2fbe83ed034dde18fd6071.pack) is corrupt
remote: aborting due to possible repository corruption on the remote side.
error: git upload-pack: git-pack-objects died with error.
fatal: git upload-pack: aborting due to possible repository corruption on the remote side.
fatal: early EOF
Gitea: Internal error
fatal: fetch-pack: invalid index-pack output
```

**服务端处理**
```sh
[git@localhost pack]$ git -c core.bigFileThreshold=256m repack -A -d -F -f
Enumerating objects: 4455, done.
Counting objects: 100% (4455/4455), done.
Delta compression using up to 2 threads
Compressing objects: 100% (4234/4234), done.
Writing objects: 100% (4455/4455), done.
Building bitmaps: 100% (40/40), done.
Total 4455 (delta 1565), reused 0 (delta 0), pack-reused 0
```

服务端处理完毕后，客户端就可以正常clone了。

## 参考
- https://kscherer.github.io/git/2021/02/11/update-git-server-option-bigfilethreshold
- https://prabuselva.github.io/linux/hacks/git-remote-malloc-error/
