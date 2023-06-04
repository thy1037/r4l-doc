# qemu + linux6.1
linux 6.1 是第一个增加 Rust 驱动开发的 TLS 版本。

本章使用 qemu 内 express-a9 模拟器将 linux 6.1 运行起来。

## qemu
[官网](https://www.qemu.org/download/)

我使用的 Debian 仓库自带的版本，直接 apt 安装即可，版本：QEMU emulator version 5.2.0 (Debian 1:5.2+dfsg-11+deb11u2)

也可在官网下载最新版本

## 编译器
[arm-linux-gnueabihf-7.5](https://releases.linaro.org/components/toolchain/binaries/7.5-2019.12/arm-linux-gnueabihf/)

根据开发环境，我下载的 gcc-linaro-7.5.0-2019.12-x86_64_arm-linux-gnueabihf.tar.xz。解压后将 gcc-linaro-7.5.0-2019.12-x86_64_arm-linux-gnueabihf/bin 添加进环境变量，便于后续使用。

## linux 内核
[官方1](https://www.kernel.org/) [官方2](https://mirrors.edge.kernel.org/pub/linux/kernel/) [国内镜像](https://mirror.bjtu.edu.cn/kernel/linux/kernel/)

我选择的 linux-6.1 最高版本 linux-6.1.21，下载、解压后进入目录。

1. 导出环境变量：
```bash
export ARCH=arm
export CROSS_COMPILE=arm-linux-gnueabihf-
```
如果没将编译器目录添加进环境变量，还需将编译器可执行文件的完整目录导出：
```bash
export PATH=$PATH:/.../gcc-linaro-7.5.0-2019.12-x86_64_arm-linux-gnueabihf/bin
```

2. 生成默认配置
```bash
make vexpress_defconfig
```

3. 编译
```bash
make zImage dtbs -j4
```

编译结束后可以得到:

    arch/arm/boot/zImage
    arch/arm/boot/dts/vexpress-v2p-ca9.dtb

4. 修改配置
为了方便修改 Makefile 文件：
```makefile
ARCH          ?= arm
CROSS_COMPILE  = arm-linux-gnueabihf-
```

5. 运行
```bash
qemu-system-arm -M vexpress-a9 -m 256M -nographic -kernel zImage -dtb dts/vexpress-v2p-ca9.dtb
```

其中： <br>
**-M**：指定 qemu 模拟的设备 <br>
**-m**：指定内存大小 <br>
**-nographic**：无图形化界面 <br>
**-kernel**：指定内核 <br>
**-dtb**：指定设备树

有如下输出：
```bash
Booting Linux on physical CPU 0x0
Linux version 6.1.21 (ehe@debian) (arm-linux-gnueabihf-gcc (Linaro GCC 7.5-2019.12) 7.5.0, GNU ld (Linaro_Binutils-2019.12) 2.28.2.20170706) #1 SMP Tue Apr  4 07:12:46 CST 2023
...
---[ end Kernel panic - not syncing: VFS: Unable to mount root fs on unknown-block(0,0) ]---
```
可以看到内核已经运行起来，内核、编译器版本、时间等信息也都显示正确，但最后出现了一个 panic，提示无法挂载根文件系统。

## 根文件系统
linux 属于宏内核，运行依赖文件系统，需要在启动阶段就加载一个文件系统，将之称为**根文件系统（root fs）**，之后从文件系统中再继续运行其它程序，完成系统的完整启动。

根文件系统的制作方法和工具，广泛使用的3种：
* busybox
* buildroot
* yocto

这里使用 **busybox**，操作最简单，也最精简。

[官网](https://www.busybox.net/)下载合适的版本，我这里选择当前最新的稳定版本 BusyBox 1.34.1

1. 配置
解压后进入目录，依旧保持之前环境变量，并打开配置界面：
```bash
export ARCH=arm
export CROSS_COMPILE=arm-linux-gnueabihf-
make menuconfig
```
为了保持简单，使用静态库编译：
```
Settings --->
    --- Build Options
    [*] Build static binary (no shared libs)
```

2. 编译并安装
```bash
make
make install
```

因为没有指定 `CONFIG_PREFIX`，默认安装路径为 busybox 下的 `_install` 目录。

3. 静态创建设备文件
```bash
cd _install
mkdir dev
cd dev
mknod -m 666 tty1 c 4 1
mknod -m 666 tty2 c 4 2
mknod -m 666 tty3 c 4 3
mknod -m 666 tty4 c 4 4
mknod -m 666 console c 5 1
mknod -m 666 null c 1 3
```

4. 制作 root fs 镜像
本质是将一块固定大小的存储空间，格式化为某种格式的文件系统，再将 `_install` 目录内的所有文件都拷贝进去，以下为虚拟设备和实体设备的对比：
a. 创建
```bash
dd if=/dev/zero of=rootfs.ext4 bs=1M count=16
```
其中： <br>
**if**：指定输入文件，这里用 `/dev/zero` 设备，即输入数据全是 0 <br>
**of**：指定输出文件，这里是 `rootfs.ext4`，`.ext4`只是普通字符串，在文件名种说明文件格式 <br>
**bs**：指定块大小，这里是 `1M` <br>
**count**：指定块数量，这里是 `16`，即最终创建的存储空间为 `16M`

> 对比实体设备的购买SD

b. 格式化
```bash
sudo mkfs.ext4 rootfs.ext4
```

> 对比实体设备的将SD卡格式化为 ext4 格式。

c. 挂载
```bash
mkdir rootfs
sudo mount -t ext4 rootfs.ext4 rootfs -o loop
```
创建一个目录 `rootfs` 将 rootfs.ext4 挂载上去。

> 对比实体设备的插入SD卡，在 /mnt 出现新的存储设备

d. 复制
```bash
sudo cp -rf _install/* rootfs
```

> 对比实体设备，就是将文件复制进SD卡

e. 卸载
```bash
sudo unmount rootfs
```

> 对比实体设备的拔除SD卡

这样 rootfs.ext4 镜像内即有 _install 目录下的所有内容

## 运行
```bash
qemu-system-arm -M vexpress-a9 -m 256M -nographic -kernel zImage -dtb dts/vexpress-v2p-ca9.dtb -sd rootfs.ext4 -append "root=/dev/mmcblk0 rw console=ttyAMA0"
```
其中： <br>
**-sd**：指定存储设备为SD，并指定SD镜像为rootfs.ext4 <br>
**-append**：添加附加命令，指定root fs 目录，指定控制台

```bash
Booting Linux on physical CPU 0x0
Linux version 6.1.21 (ehe@debian) (arm-linux-gnueabihf-gcc (Linaro GCC 7.5-2019.12) 7.5.0, GNU ld (Linaro_Binutils-2019.12) 2.28.2.20170706) #1 SMP Tue Apr  4 07:12:46 CST 2023
...
Please press Enter to activate this console.
/ # 
```
这样就完成里完整的启动，可以执行 busybox 自带的命令。


