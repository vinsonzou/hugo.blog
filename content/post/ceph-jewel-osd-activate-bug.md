+++
date = "2016-07-29T00:29:29+08:00"
description = ""
tags = ["ceph"]
title = "ceph集群jewel版本部署osd激活权限报错"

+++

**环境**

ceph version 10.2.2 (45107e21c568dd033c2f0a3107dec8f0b0e58374)</br>
ceph-deploy 1.5.34

ceph集群jewel版本部署过程中执行osd激活操作如下
```sh
ceph-deploy osd activate ceph13:/dev/sdb1:/dev/sda2
```

报错内容如下
```sh
[2016-07-29 00:05:19,106][ceph_deploy.conf][DEBUG ] found configuration file at: /root/.cephdeploy.conf
[2016-07-29 00:05:19,107][ceph_deploy.cli][INFO  ] Invoked (1.5.34): /bin/ceph-deploy osd activate ceph13:/dev/sdb1:/dev/sda2
[2016-07-29 00:05:19,107][ceph_deploy.cli][INFO  ] ceph-deploy options:
[2016-07-29 00:05:19,108][ceph_deploy.cli][INFO  ]  username                      : None
[2016-07-29 00:05:19,108][ceph_deploy.cli][INFO  ]  verbose                       : False
[2016-07-29 00:05:19,108][ceph_deploy.cli][INFO  ]  overwrite_conf                : False
[2016-07-29 00:05:19,108][ceph_deploy.cli][INFO  ]  subcommand                    : activate
[2016-07-29 00:05:19,108][ceph_deploy.cli][INFO  ]  quiet                         : False
[2016-07-29 00:05:19,108][ceph_deploy.cli][INFO  ]  cd_conf                       : <ceph_deploy.conf.cephdeploy.Conf instance at 0x26dcb90>
[2016-07-29 00:05:19,109][ceph_deploy.cli][INFO  ]  cluster                       : ceph
[2016-07-29 00:05:19,109][ceph_deploy.cli][INFO  ]  func                          : <function osd at 0x26d0320>
[2016-07-29 00:05:19,109][ceph_deploy.cli][INFO  ]  ceph_conf                     : None
[2016-07-29 00:05:19,109][ceph_deploy.cli][INFO  ]  default_release               : False
[2016-07-29 00:05:19,109][ceph_deploy.cli][INFO  ]  disk                          : [('ceph13', '/dev/sdb1', '/dev/sda2')]
[2016-07-29 00:05:19,110][ceph_deploy.osd][DEBUG ] Activating cluster ceph disks ceph13:/dev/sdb1:/dev/sda2
[2016-07-29 00:05:19,217][ceph13][DEBUG ] connected to host: ceph13
[2016-07-29 00:05:19,218][ceph13][DEBUG ] detect platform information from remote host
[2016-07-29 00:05:19,258][ceph13][DEBUG ] detect machine type
[2016-07-29 00:05:19,264][ceph13][DEBUG ] find the location of an executable
[2016-07-29 00:05:19,266][ceph_deploy.osd][INFO  ] Distro info: CentOS Linux 7.2.1511 Core
[2016-07-29 00:05:19,266][ceph_deploy.osd][DEBUG ] activating host ceph13 disk /dev/sdb1
[2016-07-29 00:05:19,266][ceph_deploy.osd][DEBUG ] will use init type: systemd
[2016-07-29 00:05:19,266][ceph13][DEBUG ] find the location of an executable
[2016-07-29 00:05:19,270][ceph13][INFO  ] Running command: /usr/sbin/ceph-disk -v activate --mark-init systemd --mount /dev/sdb1
[2016-07-29 00:05:19,598][ceph13][WARNING] main_activate: path = /dev/sdb1
[2016-07-29 00:05:19,598][ceph13][WARNING] get_dm_uuid: get_dm_uuid /dev/sdb1 uuid path is /sys/dev/block/8:17/dm/uuid
[2016-07-29 00:05:19,598][ceph13][WARNING] command: Running command: /sbin/blkid -o udev -p /dev/sdb1
[2016-07-29 00:05:19,630][ceph13][WARNING] command: Running command: /sbin/blkid -p -s TYPE -o value -- /dev/sdb1
[2016-07-29 00:05:19,638][ceph13][WARNING] command: Running command: /usr/bin/ceph-conf --cluster=ceph --name=osd. --lookup osd_mount_options_xfs
[2016-07-29 00:05:19,803][ceph13][WARNING] command: Running command: /usr/bin/ceph-conf --cluster=ceph --name=osd. --lookup osd_fs_mount_options_xfs
[2016-07-29 00:05:19,967][ceph13][WARNING] mount: Mounting /dev/sdb1 on /var/lib/ceph/tmp/mnt.K5WBsO with options noatime,inode64
[2016-07-29 00:05:19,967][ceph13][WARNING] command_check_call: Running command: /usr/bin/mount -t xfs -o noatime,inode64 -- /dev/sdb1 /var/lib/ceph/tmp/mnt.K5WBsO
[2016-07-29 00:05:19,983][ceph13][WARNING] command: Running command: /sbin/restorecon /var/lib/ceph/tmp/mnt.K5WBsO
[2016-07-29 00:05:19,991][ceph13][WARNING] activate: Cluster uuid is f2694afb-b5fc-4225-982a-342c4a1ca389
[2016-07-29 00:05:19,991][ceph13][WARNING] command: Running command: /usr/bin/ceph-osd --cluster=ceph --show-config-value=fsid
[2016-07-29 00:05:20,156][ceph13][WARNING] activate: Cluster name is ceph
[2016-07-29 00:05:20,156][ceph13][WARNING] activate: OSD uuid is 684effdc-67ae-4f54-a9b8-a113f4a8f0cc
[2016-07-29 00:05:20,156][ceph13][WARNING] activate: OSD id is 0
[2016-07-29 00:05:20,156][ceph13][WARNING] activate: Initializing OSD...
[2016-07-29 00:05:20,157][ceph13][WARNING] command_check_call: Running command: /usr/bin/ceph --cluster ceph --name client.bootstrap-osd --keyring /var/lib/ceph/bootstrap-osd/ceph.keyring mon getmap -o /var/lib/ceph/tmp/mnt.K5WBsO/activate.monmap
[2016-07-29 00:05:20,572][ceph13][WARNING] got monmap epoch 1
[2016-07-29 00:05:20,572][ceph13][WARNING] command: Running command: /usr/bin/timeout 300 ceph-osd --cluster ceph --mkfs --mkkey -i 0 --monmap /var/lib/ceph/tmp/mnt.K5WBsO/activate.monmap --osd-data /var/lib/ceph/tmp/mnt.K5WBsO --osd-journal /var/lib/ceph/tmp/mnt.K5WBsO/journal --osd-uuid 684effdc-67ae-4f54-a9b8-a113f4a8f0cc --keyring /var/lib/ceph/tmp/mnt.K5WBsO/keyring --setuser ceph --setgroup ceph
[2016-07-29 00:05:20,686][ceph13][WARNING] mount_activate: Failed to activate
[2016-07-29 00:05:20,686][ceph13][WARNING] unmount: Unmounting /var/lib/ceph/tmp/mnt.K5WBsO
[2016-07-29 00:05:20,687][ceph13][WARNING] command_check_call: Running command: /bin/umount -- /var/lib/ceph/tmp/mnt.K5WBsO
[2016-07-29 00:05:21,052][ceph13][WARNING] Traceback (most recent call last):
[2016-07-29 00:05:21,052][ceph13][WARNING]   File "/usr/sbin/ceph-disk", line 9, in <module>
[2016-07-29 00:05:21,052][ceph13][WARNING]     load_entry_point('ceph-disk==1.0.0', 'console_scripts', 'ceph-disk')()
[2016-07-29 00:05:21,052][ceph13][WARNING]   File "/usr/lib/python2.7/site-packages/ceph_disk/main.py", line 4994, in run
[2016-07-29 00:05:21,053][ceph13][WARNING]     main(sys.argv[1:])
[2016-07-29 00:05:21,053][ceph13][WARNING]   File "/usr/lib/python2.7/site-packages/ceph_disk/main.py", line 4945, in main
[2016-07-29 00:05:21,053][ceph13][WARNING]     args.func(args)
[2016-07-29 00:05:21,053][ceph13][WARNING]   File "/usr/lib/python2.7/site-packages/ceph_disk/main.py", line 3299, in main_activate
[2016-07-29 00:05:21,053][ceph13][WARNING]     reactivate=args.reactivate,
[2016-07-29 00:05:21,054][ceph13][WARNING]   File "/usr/lib/python2.7/site-packages/ceph_disk/main.py", line 3056, in mount_activate
[2016-07-29 00:05:21,054][ceph13][WARNING]     (osd_id, cluster) = activate(path, activate_key_template, init)
[2016-07-29 00:05:21,054][ceph13][WARNING]   File "/usr/lib/python2.7/site-packages/ceph_disk/main.py", line 3232, in activate
[2016-07-29 00:05:21,054][ceph13][WARNING]     keyring=keyring,
[2016-07-29 00:05:21,054][ceph13][WARNING]   File "/usr/lib/python2.7/site-packages/ceph_disk/main.py", line 2725, in mkfs
[2016-07-29 00:05:21,055][ceph13][WARNING]     '--setgroup', get_ceph_group(),
[2016-07-29 00:05:21,055][ceph13][WARNING]   File "/usr/lib/python2.7/site-packages/ceph_disk/main.py", line 2672, in ceph_osd_mkfs
[2016-07-29 00:05:21,055][ceph13][WARNING]     raise Error('%s failed : %s' % (str(arguments), error))
[2016-07-29 00:05:21,055][ceph13][WARNING] ceph_disk.main.Error: Error: ['ceph-osd', '--cluster', 'ceph', '--mkfs', '--mkkey', '-i', '0', '--monmap', '/var/lib/ceph/tmp/mnt.K5WBsO/activate.monmap', '--osd-data', '/var/lib/ceph/tmp/mnt.K5WBsO', '--osd-journal', '/var/lib/ceph/tmp/mnt.K5WBsO/journal', '--osd-uuid', '684effdc-67ae-4f54-a9b8-a113f4a8f0cc', '--keyring', '/var/lib/ceph/tmp/mnt.K5WBsO/keyring', '--setuser', 'ceph', '--setgroup', 'ceph'] failed : 2016-07-29 00:05:20.672440 7f46517dd800 -1 filestore(/var/lib/ceph/tmp/mnt.K5WBsO) mkjournal error creating journal on /var/lib/ceph/tmp/mnt.K5WBsO/journal: (13) Permission denied
[2016-07-29 00:05:21,056][ceph13][WARNING] 2016-07-29 00:05:20.672462 7f46517dd800 -1 OSD::mkfs: ObjectStore::mkfs failed with error -13
[2016-07-29 00:05:21,056][ceph13][WARNING] 2016-07-29 00:05:20.672526 7f46517dd800 -1  ** ERROR: error creating empty object store in /var/lib/ceph/tmp/mnt.K5WBsO: (13) Permission denied
[2016-07-29 00:05:21,056][ceph13][WARNING]
[2016-07-29 00:05:21,057][ceph13][ERROR ] RuntimeError: command returned non-zero exit status: 1
[2016-07-29 00:05:21,057][ceph_deploy][ERROR ] RuntimeError: Failed to execute command: /usr/sbin/ceph-disk -v activate --mark-init systemd --mount /dev/sdb1
```

**解决办法:**

将ceph集群需要使用的所有磁盘权限，所属用户、用户组改给ceph
```sh
chown ceph:ceph /dev/sdb1
```

**问题延伸:**

此问题本次修复后，系统重启磁盘权限会被修改回，导致osd服务无法正常启动，这个权限问题很坑，每次系统启动自动修改磁盘权限。

查找ceph资料，发现这其实是一个bug，社区暂未解决。

http://tracker.ceph.com/issues/13833
