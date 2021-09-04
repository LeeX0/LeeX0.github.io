---
title: DPDK配置测试
tags:
  - - DPDK
  - - 数据转发
categories:
  - [项目组, DPDK]
abbrlink: 7214eebe
date: 2021-06-05 14:29:37
---

> 配置信息：
>
> VMware® Workstation 16 Pro: 16.1.1 build-17801498
>
> Ubuntu-16.04.6: Linux ubuntu 4.15.0-142-generic
>
> CPU: 2个处理器，每个处理器2个内核
>
> 内存: 4GB
>
> 网卡: 网络适配器-NAT(ens33)，网络适配器2-Host-only(ens38)，网络适配器3-Host-only(ens39)
>
> [DPDK](http://core.dpdk.org/download/): DPDK-17.11.10(LTS)

### 1. DPDK下载

```bash
wget http://fast.dpdk.org/rel/dpdk-17.11.10.tar.xz
tar -xvf dpdk-17.11.10.tar.xz
```

### 2. 编译前修改

> DPDK在虚拟机中实验，本例使用vmware添加的虚拟网卡，会有兼容性问题。
>
> `EAL:Error reading from file descriptor xx: Input/output error`
>
> 修改`/dpdk-stable-17.11.10/lib/librte_eal/linuxapp/igb_uio/igb_uio.c`以下内容，跳过检测。

```c
/* fall back to INTX */
	case RTE_INTR_MODE_LEGACY:
		if (pci_intx_mask_supported(dev）) {
			dev_dbg(&dev->dev, "using INTX");
			udev->info.irq_flags = IRQF_SHARED | IRQF_NO_THREAD;
			udev->info.irq = dev->irq;
			udev->mode = RTE_INTR_MODE_LEGACY;
			break;
		}
		dev_notice(&dev->dev, "PCI INTX mask not supported
               
// 修改为：if判断中增加true使其为真
               
	/* fall back to INTX */
	case RTE_INTR_MODE_LEGACY:
		if (pci_intx_mask_supported(dev)||true) {
			dev_dbg(&dev->dev, "using INTX");
			udev->info.irq_flags = IRQF_SHARED | IRQF_NO_THREAD;
			udev->info.irq = dev->irq;
			udev->mode = RTE_INTR_MODE_LEGACY;
			break;
		}
		dev_notice(&dev->dev, "PCI INTX mask not supported\
```

### 3. 配置环境变量-编译-配置其他选项

```bash
export RTE_SDK=<dpdk主目录> 
# export RTE_SDK=/opt/dpdk-stable-17.11.10
export RTE_TARGET=x86_64-native-linuxapp-gcc
```

较早版本的DPDK都提供了一个安装脚本`/dpdk-stable-17.11.10/usertools/dpdk-setup.sh`，方便完成编译、设置、测试等内容。

```bash
./dpdk-setup.sh

# 选择[14]编译DPDK
[14] x86_64-native-linuxapp-gcc

# 设置大页内存，本次设定的512
[21] Setup hugepage mappings for NUMA systems
# 查看大页内存
[28] List hugepage info from /proc/meminfo

# 载入IGB UIO模块
[17] Insert IGB UIO module

# 查看当前网卡绑定情况
[22] Display current Ethernet/Crypto device settings
# 若需要绑定到DPDK的网卡处于*Active*状态，需要先down掉

# 退出脚本，down网卡
[34] Exit Script
```

```bash
# 查看网卡信息，需要down那些网卡
ifconfig
# 本例down另外配置的ens38与ens39网卡
ifconfig ens38 down
ifconfig ens39 down
```

```bash
# 重新进入脚本绑定网卡到DPDK
./dpdk-setup.sh
[22] Display current Ethernet/Crypto device settings
# 输入网卡前的描述符进行绑定，本例ens38对应02:06.0，ens39对应02:07.0
02:06.0
02:07.0

# 查看当前网卡绑定情况
[22] Display current Ethernet/Crypto device settings
# 此时应该能看到刚才设定的两张网卡已设定到DPDK-compatible driver
```

### 4. 简单测试

完成以上配置之后DPDK配置基本已经完成，可以仍在安装脚本中直接进行测试。

```bash
[27] Run testpmd application in interactive mode ($RTE_TARGET/app/testpmd)

bitmask:f
Launching app
EAL: Detected 4 lcore(s)
EAL: No free hugepages reported in hugepages-1048576kB
EAL: Probing VFIO support...
EAL: PCI device 0000:02:01.0 on NUMA socket -1
EAL:   Invalid NUMA socket, default to 0
EAL:   probe driver: 8086:100f net_e1000_em
EAL: PCI device 0000:02:06.0 on NUMA socket -1
EAL:   Invalid NUMA socket, default to 0
EAL:   probe driver: 8086:100f net_e1000_em
EAL: PCI device 0000:02:07.0 on NUMA socket -1
EAL:   Invalid NUMA socket, default to 0
EAL:   probe driver: 8086:100f net_e1000_em
Interactive-mode selected
USER1: create a new mbuf pool <mbuf_pool_socket_0>: n=171456, size=2176, socket=0
Configuring Port 0 (socket 0)
Port 0: 00:0C:29:3E:A9:4F
Configuring Port 1 (socket 0)
Port 1: 00:0C:29:3E:A9:59
Checking link statuses...
Done

testpmd> start
io packet forwarding - ports=2 - cores=1 - streams=2 - NUMA support enabled, MP over anonymous pages disabled
Logical Core 1 (socket 0) forwards packets on 2 streams:
  RX P=0/Q=0 (socket 0) -> TX P=1/Q=0 (socket 0) peer=02:00:00:00:00:01
  RX P=1/Q=0 (socket 0) -> TX P=0/Q=0 (socket 0) peer=02:00:00:00:00:00

  io packet forwarding packets/burst=32
  nb forwarding cores=1 - nb forwarding ports=2
  port 0:
  CRC stripping enabled
  RX queues=1 - RX desc=128 - RX free threshold=0
  RX threshold registers: pthresh=0 hthresh=0  wthresh=0
  TX queues=1 - TX desc=512 - TX free threshold=0
  TX threshold registers: pthresh=0 hthresh=0  wthresh=0
  TX RS bit threshold=0 - TXQ flags=0x0
  port 1:
  CRC stripping enabled
  RX queues=1 - RX desc=128 - RX free threshold=0
  RX threshold registers: pthresh=0 hthresh=0  wthresh=0
  TX queues=1 - TX desc=512 - TX free threshold=0
  TX threshold registers: pthresh=0 hthresh=0  wthresh=0
  TX RS bit threshold=0 - TXQ flags=0x0

testpmd> stop
Telling cores to stop...
Waiting for lcores to finish...
  ---------------------- Forward statistics for port 0  ----------------------
  RX-packets: 7194           RX-dropped: 0             RX-total: 7194
  TX-packets: 6886           TX-dropped: 0             TX-total: 6886
  ----------------------------------------------------------------------------

  ---------------------- Forward statistics for port 1  ----------------------
  RX-packets: 6886           RX-dropped: 0             RX-total: 6886
  TX-packets: 7194           TX-dropped: 0             TX-total: 7194
  ----------------------------------------------------------------------------

  +++++++++++++++ Accumulated forward statistics for all ports+++++++++++++++
  RX-packets: 14080          RX-dropped: 0             RX-total: 14080
  TX-packets: 14080          TX-dropped: 0             TX-total: 14080
  ++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
Done.

testpmd> quit
Stopping port 0...
Stopping ports...
Done

Stopping port 1...
Stopping ports...
Done

Shutting down port 0...
Closing ports...
Done

Shutting down port 1...
Closing ports...
Done

Bye...
```

### 其他版本相关

> 和上述内容相似，在虚拟机中使用igb_uio驱动时仍可能遇到网卡不兼容问题，在你的驱动编译前修改并调过验证。

```bash
### 依赖检查
gcc -v
# gcc (version 4.9+) 
pkg-config --version
# pkg-config 0.27+
uname -r
# Kernel version >= 4.4
ldd --version
# glibc >= 2.7
grep -i huge /boot/config-4.15.0-142-generic
# Output
CONFIG_HUGETLBFS=y
CONFIG_HUGETLB_PAGE=y
pip3 install meson ninja

### dpdk下载
wget http://fast.dpdk.org/rel/dpdk-20.11.1.tar.xz
tar xJf dpdk-20.11.1.tar.xz
cd dpdk-stable-20.11.1
# dpdk编译安装
meson -Dexamples=all build
cd build
ninja install
	
### 配置大页内存
mkdir /mnt/huge
echo 512 > /sys/devices/system/node/node0/hugepages/hugepages-2048kB/nr_hugepages
mount -t hugetlbfs nodev /mnt/huge
grep -i huge /proc/meminfo
# 透明大页配置
echo always >/sys/kernel/mm/transparent_hugepage/enabled
echo madvise >/sys/kernel/mm/transparent_hugepage/enabled
echo never >/sys/kernel/mm/transparent_hugepage/enabled

### 自动挂载
vim /etc/fstab
#添加以下内容
nodev /mnt/huge hugetlbfs pagesize=2MB 0 0
#如果是1GB 则nodev /mnt/huge hugetlbfs pagesize=1GB 0 0

### 编译产生问题可能由于缺少以下组件
sudo apt-get install numactl
sudo apt-get install libnuma-dev

```

---

> 参考：
>
> http://doc.dpdk.org/guides-20.11/
>
> https://blog.csdn.net/qwe13182912113/article/details/79260331?utm_medium=distribute.pc_relevant.none-task-blog-2~default~BlogCommendFromMachineLearnPai2~default-1.control&dist_request_id=1332048.21311.16195122750681543&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2~default~BlogCommendFromMachineLearnPai2~default-1.control
>
> https://zhuanlan.zhihu.com/p/368045406
