---
title: DPDK17挂载与卸载脚本
tags:
  - - DPDK
  - - 数据转发
categories:
  - [项目组, DPDK]
abbrlink: '655e2191'
date: 2021-06-07 15:34:18
---

> [DPDK](http://core.dpdk.org/download/): DPDK-17.11.10(LTS)
>
> 每次相关设置改变需要对DPDK重启时，都要进行许多配置操作较为麻烦。
>
> 参考`DPDK/usertools/dpdk-setup.sh`写成DPDK的挂载与卸载脚本方便使用。
>
> 其中`RTE_SDK`、`RTE_TARGET`、网卡地址等参数根据自己实际情况填入，不再写交互式脚本（否则与`dpdk-setup.sh`无异）。

`dpdk-mount.rc`

```bash
#!/bin/sh

export RTE_SDK=/opt/dpdk-stable-17.11.10
export RTE_TARGET=x86_64-native-linuxapp-gcc

/sbin/modprobe uio
/sbin/insmod $RTE_SDK/$RTE_TARGET/kmod/igb_uio.ko

mkdir -p /mnt/huge
mount -t hugetlbfs nodev /mnt/huge
echo 512 > /sys/devices/system/node/node0/hugepages/hugepages-2048kB/nr_hugepages

ifconfig ens38 down
ifconfig ens39 down

${RTE_SDK}/usertools/dpdk-devbind.py -b igb_uio 02:06.0
${RTE_SDK}/usertools/dpdk-devbind.py -b igb_uio 02:07.0


grep -i huge /proc/meminfo
${RTE_SDK}/usertools/dpdk-devbind.py --status
```

`dpdk-rm.rc`

```bash
#!/bin/sh

${RTE_SDK}/usertools/dpdk-devbind.py -b e1000 02:06.0
${RTE_SDK}/usertools/dpdk-devbind.py -b e1000 02:07.0

echo 0 > /sys/devices/system/node/node0/hugepages/hugepages-2048kB/nr_hugepages

umount /mnt/huge
rm -R /mnt/huge

/sbin/rmmod igb_uio

grep -i huge /proc/meminfo  
${RTE_SDK}/usertools/dpdk-devbind.py --status

unset RTE_SDK
unset RTE_TARGET
```

