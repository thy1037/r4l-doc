# 最简驱动体验
如果在 Docker 中做开发，先参考 [Docker 安装](./env_docker.md) 和 [Docker 配置](./env_docker_config.md) 完成环境的搭建，其它内容参考 [qemu + linux6.1](./env_01.md)，这边只介绍 rust 相关的新增内容。

## 编译准备
这边选用当前 linux 6.1 的最高版本 [6.1.25](https://cdn.kernel.org/pub/linux/kernel/v6.x/linux-6.1.25.tar.xz)。

下载、解压后进入，查看 `Documentation/rust/quick-start.rst` 可以看到开启内核里Rust支持的一些先决条件：
```bash
# 1. 指定 rustc 版本：
rustup override set $(scripts/min-tool-version.sh rustc)

# 2. 添加 rust 源码
rustup component add rust-src

# 3. libclang：从 https://releases.llvm.org/download.html 下载最新编译好的内容，将 bin 文件夹加入环境变量

# 4. 指定 binggen 版本：
cargo install --locked --version $(scripts/min-tool-version.sh bindgen) bindgen

# 5. 其余内容在安装 rust 时都已自动安装

# 6. 检查依赖是否满足
make LLVM=1 rustavailable
```

## 内核配置
```bash
# 1. 选择基本环境，当前版本内核支持的并不宽泛，x86_64 和 arm64（aarch64）的限制少： 
make x86_64_defconfig

# 2.1 General setup ->
#       [*] Rust support
# 2.2 Kernel hacking ->
#       [*] Sample kernel code ->
#           [*] Rust samples ->
#               [*] Minimal
make LLVM=1 menuconfig

# 3. 编译
make LLVM=1
```
其中 2.1 是使能内核的 Rust 支持， 2.2 是将内核自带的一个最简模块示例代码编译进内核镜像，启动内核是可以自动加载。

## 根文件系统制作
在 [qemu + linux6.1](./env_01.md) 中介绍了根文件系统的详细步骤，这里因为 linux 选择了 `x86_64_defconfig` 使用另一种简单的方式，将其写为了脚本，方便使用
```bash
#!/bin/sh
busybox_folder="/download/busybox-1.34.1"
rootfs="rootfs"
rootfs_img=$PWD"/rootfs.img"

echo $base_path
if [ ! -d $rootfs ]; then #判断文件是否存在
	mkdir $rootfs
fi
cp $busybox_folder/_install/*  $rootfs/ -rf
cd $rootfs
if [ ! -d proc ] && [ ! -d sys ] && [ ! -d dev ] && [ ! -d etc/init.d ]; then
	mkdir proc sys dev etc etc/init.d
fi
 
if [ -f etc/init.d/rcS ]; then
	rm etc/init.d/rcS
fi
echo "#!/bin/sh" > etc/init.d/rcS
echo "mount -t proc none /proc" >> etc/init.d/rcS
echo "mount -t sysfs none /sys" >> etc/init.d/rcS
echo "/sbin/mdev -s" >> etc/init.d/rcS
chmod +x etc/init.d/rcS
if [ -f $rootfs_img ]; then
	rm $rootfs_img
fi
find . | cpio -o --format=newc > $rootfs_img
```

执行脚本后即可在当前目录得到 `rootfs.img` 文件。

## 启动内核
为了方便使用，将启动命令写进了 Makefile：
```makefile
KERNEL_IMG = /download/linux-6.1.25/arch/x86_64/boot/bzImage
QEMU = qemu-system-x86_64
ROOTFS_IMG = rootfs.img

default:
	$(QEMU) -kernel $(KERNEL_IMG) -append "root=/dev/ram rdinit=sbin/init console=ttyS0" -nographic -initrd $(ROOTFS_IMG)

```
执行 `make` 指令即可启动内核：
```
[    0.000000] Linux version 6.2.0 (root@5deb884f5933) (gcc (Debian 10.2.1-6) 10.2.1 20210110, GNU ld (GNU Binutils for Debian) 2.35.2) #1 SMP PREEMPT_DYNAMIC 3
[    0.000000] Command line: root=/dev/ram rdinit=sbin/init console=ttyS0
...
[    1.686751] usbhid: USB HID core driver
[    1.689337] rust_minimal: Rust minimal sample (init)
[    1.689571] rust_minimal: Am I built-in? true
[    1.692888] Initializing XFRM netlink socket
...
Please press Enter to activate this console.
/ # 
```
可以看到 [rust_minimal] 模块在挂载时的日志内容。