# linux-fujita + e1000

复现用 Rust 编写的 e1000 网卡驱动。

## Linux
使用开发者 [fujita](https://github.com/fujita/linux/tree/rust-e1000) 开发维护的 linux 内核源码，这里增加了 PCI、DMA、NIC 等网络API的抽象。

> ！注意！
>
> 如果是自行进入 github.com/fujita/linux，一定要切换到 `rust-e1000` 分支！

```bash
# 1. 选择基本环境 
make x86_64_defconfig

# 2.1 General setup ->
#       [*] Rust support
# 2.2 Kernel hacking ->
#       [*] Sample kernel code ->
#           [*] Rust samples ->
#               [*] Minimal
# 2.3 Device Driver ->
#       Network device support ->
#           Ethernet driver support ->
#               Intel devices ->
#                   <*> Intel(R) PRO/100+ support
#                   <M> Intel(R) PRO/1000 Gigabit Ethernet support
#                   <*> Intel(R) PRO/1000 PCI-Express Gigabit Ethernet support
make LLVM=1 menuconfig

# 3. 编译
make LLVM=1 -j8
```
其中：

2.3 是将原本默认选择的e1000网卡驱动选择去除，这里为了避免其它编译依赖问题，查看 Kconfig 后将 `Intel(R) PRO/1000 Gigabit Ethernet support` 选为 M 模式。

## e1000-fujita
这是开发者 [fujita](https://github.com/fujita/rust-e1000) 的 e1000 驱动，如其 README 所述：`This driver works on QEMU, howerver nothing else works.`，纯粹为了验证可以编译ko模块。

1. 下载、传入 Docker 后进入目录，执行 `make KDIR=/your/linux-rust-e1000/path LLVM=1`，其中`/your/linux-rust-e1000/path` 即[前面](#linux)编译的 linux 目录，编译完成后得到 `rust_e1000.ko` 文件，[重新制作](./rust_01.md#根文件系统制作)根文件系统镜像

2. 挂载模块、[启动模块](./rust_01.md#启动内核)
```
[    0.000000] Linux version 6.1.0-rc1 (root@183b0e7b45ae) (Debian clang version 11.0.1-2, LLD 11.0.1) #1 SMP PREEMPT_DYNAMIC Wed May 10 13:05:54 UTC 2023
...
[    1.711355] rust_minimal: Rust minimal sample (init)
[    1.711671] rust_minimal: Am I built-in? true
Please press Enter to activate this console. 
/ # ls
bin                linuxrc            root               sbin
dev                proc               rust_e1000.ko      sys
etc                usr
/ # insmod rust_e1000.ko 
[  422.320006] rust_e1000: loading out-of-tree module taints kernel.
[  422.327016] insmod (79) used greatest stack depth: 13320 bytes left
```

可以看到模块被加载的日志输出。

## e1000-myrfy001
这是开发者 [myrfy001](https://github.com/myrfy001/r4l-e1000) 的 e1000 驱动，具备一定的网络能力，如 `ping`。

参考 [e1000-fujita](#e1000-fujita) 完成驱动模块的加载，或者使用以下脚本：
```bash
set -e

# Build the kernel module
make -C /download/linux-rust-e1000/ M=$PWD LLVM=1 CLIPPY=1

# Copy the build kernel module to the rootfs folder, there should be a usable busybox at `rootfs` folder
cp ./r4l_e1000_demo.ko ../rootfs

# Build the rootfs image
pushd ../rootfs && \
find . | cpio -o --format=newc > ../rootfs.img && \
popd


# Run QEMU to test the kernel, the network traffic will be dumped to `dump.dat` and you can use wireshark to explore the result.
qemu-system-x86_64 \
-netdev "user,id=eth0" \
-device "e1000,netdev=eth0" \
-object "filter-dump,id=eth0,netdev=eth0,file=dump.dat" \
-kernel /download/linux-rust-e1000/arch/x86/boot/bzImage \
-append "root=/dev/ram rdinit=sbin/init console=ttyS0" \
-nographic \
-initrd ../rootfs.img
```

启动后执行以下命令：
```bash
# 1. 加载 ko 驱动模块
insmod r4l_e1000_demo.ko

# 2. 使能网卡
ip link set eth0 up

# 3. 设置 ip 和掩码
ip addr add 10.0.2.15/255.255.255.0 dev eth0

# 4. 10.0.2.2 是 qemu 的内部网关(防火墙、DHCP 服务器)
ping 10.0.2.2

# 5. 192.168.0.113 是运行 docker 的宿主机，此时还 ping 不通，显示网络无法到达
ping 192.168.0.113

# 6. 设置路由（网关）
ip route add default via 10.0.2.2

# 7. 192.168.0.113 此时可以 ping 通
ping 192.168.0.113

# 8. 百度 ip 110.242.68.66
ping 110.242.68.66
```

输出如下：
```
[    0.000000] Linux version 6.1.0-rc1 (root@7384ad8b51fb) (Debian clang version 11.0.1-2, LLD 11.0.1) #1 SMP PREEMPT_DYNAMIC Wed May 10 15:39:37 UTC 2023
[    0.000000] Command line: root=/dev/ram rdinit=sbin/init console=ttyS0
...
Please press Enter to activate this console.
/ # insmod r4l_e1000_demo.ko
[   19.885701] r4l_e1000_demo: loading out-of-tree module taints kernel.
[   19.891582] r4l_e1000_demo: Rust for linux e1000 driver demo (init)
[   19.891961] r4l_e1000_demo: Rust for linux e1000 driver demo (probe): None
[   20.053107] ACPI: \_SB_.LNKC: Enabled at IRQ 11
[   20.074398] r4l_e1000_demo: Rust for linux e1000 driver demo (net device get_stats64)
[   20.076238] insmod (79) used greatest stack depth: 10792 bytes left
/ # ip link set eth0 up
[   74.903901] r4l_e1000_demo: Rust for linux e1000 driver demo (net device open)
[   74.906640] r4l_e1000_demo: Rust for linux e1000 driver demo (net device get_stats64)
[   74.907998] IPv6: ADDRCONF(NETDEV_CHANGE): eth0: link becomes ready
[   74.911179] r4l_e1000_demo: Rust for linux e1000 driver demo (net device get_stats64)
/ # ip addr add 10.0.2.15/255.255.255.0 dev eth0
[   94.348826] r4l_e1000_demo: Rust for linux e1000 driver demo (net device get_stats64)
[   94.350660] r4l_e1000_demo: Rust for linux e1000 driver demo (net device get_stats64)
/ # ping 10.0.2.2
PING 10.0.2.2 (10.0.2.2): 56 data bytes
64 bytes from 10.0.2.2: seq=0 ttl=255 time=7.832 ms
64 bytes from 10.0.2.2: seq=1 ttl=255 time=2.218 ms
64 bytes from 10.0.2.2: seq=2 ttl=255 time=2.149 ms
64 bytes from 10.0.2.2: seq=3 ttl=255 time=1.937 ms
^C
--- 10.0.2.2 ping statistics ---
4 packets transmitted, 4 packets received, 0% packet loss
round-trip min/avg/max = 1.937/3.534/7.832 ms
/ # ping 192.168.0.113
PING 192.168.0.113 (192.168.0.113): 56 data bytes
ping: sendto: Network is unreachable
/ # ip route add default via 10.0.2.2
/ # ping 192.168.0.113
PING 192.168.0.113 (192.168.0.113): 56 data bytes
64 bytes from 192.168.0.113: seq=0 ttl=255 time=0.540 ms
64 bytes from 192.168.0.113: seq=1 ttl=255 time=1.968 ms
64 bytes from 192.168.0.113: seq=2 ttl=255 time=2.016 ms
64 bytes from 192.168.0.113: seq=3 ttl=255 time=2.369 ms
^C
--- 192.168.0.113 ping statistics ---
5 packets transmitted, 5 packets received, 0% packet loss
round-trip min/avg/max = 0.540/1.822/2.369 ms
/ # ping 110.242.68.66
PING 110.242.68.66 (110.242.68.66): 56 data bytes
64 bytes from 110.242.68.66: seq=0 ttl=255 time=33.533 ms
64 bytes from 110.242.68.66: seq=1 ttl=255 time=32.218 ms
64 bytes from 110.242.68.66: seq=2 ttl=255 time=32.161 ms
64 bytes from 110.242.68.66: seq=3 ttl=255 time=31.477 ms
^C
--- 110.242.68.66 ping statistics ---
4 packets transmitted, 4 packets received, 0% packet loss
round-trip min/avg/max = 31.477/32.347/33.533 ms
```


## 笔记
1. QEMU 网络模型
```
guest (10.0.2.15)  <------>  Firewall/DHCP server <-----> Internet
                      |          (10.0.2.2)
                      |
                      ---->  DNS server (10.0.2.3)
                      |
                      ---->  SMB server (10.0.2.4)
```
详见[官方文档](https://www.qemu.org/docs/master/system/devices/net.html)

2. QEMU与宿主机之间的通信机制

QEMU提供了四种网络通信模式：TAP、user、Sockets和VDE。利用user模式可以实现虚拟机和宿主机之间的通信且较为简单易行，在这种通信模式中，虚拟机处于10.0.2.*网段，该网段通过一个NAT服务器与外界通信，NAT服务器的地址是10.0.2.2，虚拟机的IP地址从10.0.2.15开始分配。<br>
———————————————— <br>
版权声明：本文为CSDN博主「小呀小二笙」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：<a>https://blog.csdn.net/qq_38790716/article/details/85019030</a>
