#### 查看开机启动项

```shell
//查询开机启动项
systemctl list-unit-files
// 输出 UNIT FILE 对应服务名；STATE 是状态：enable 是开启启动，disable是开机不启动
UNIT FILE                                     STATE
proc-sys-fs-binfmt_misc.automount             static
dev-hugepages.mount                           static
dev-mqueue.mount                              static
proc-fs-nfsd.mount                            static
proc-sys-fs-binfmt_misc.mount                 static
sys-fs-fuse-connections.mount                 static
sys-kernel-config.mount                       static
sys-kernel-debug.mount                        static
tmp.mount                                     disabled
var-lib-nfs-rpc_pipefs.mount                  static
brandbot.path                                 disabled
cups.path                                     enabled
```

```shell
下面以nfs服务为例：

1.启动nfs服务
systemctl start nfs-server.service

2.设置开机自启动
systemctl enable nfs-server.service

3.停止开机自启动
systemctl disable nfs-server.service

4.查看服务当前状态
systemctl status nfs-server.service

5.重新启动某服务
systemctl restart nfs-server.service

6.查看所有已启动的服务
 systemctl list-unit-files | grep enabled
```

