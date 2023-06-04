# 最简模块
这一章介绍如何在 linux 源码目录外编写 `.ko` 模块，用于动态加载、卸载。

## 模块代码
依然是内核中的代码，只是将其复制出来：
```rust
// SPDX-License-Identifier: GPL-2.0

//! Rust minimal sample.

use kernel::prelude::*;

module! {
    type: RustMinimal,
    name: b"rust_minimal",
    author: b"Rust for Linux Contributors",
    description: b"Rust minimal sample",
    license: b"GPL",
}

struct RustMinimal {
    numbers: Vec<i32>,
}

impl kernel::Module for RustMinimal {
    fn init(_module: &'static ThisModule) -> Result<Self> {
        pr_info!("Rust minimal sample (init)\n");
        pr_info!("Am I built-in? {}\n", !cfg!(MODULE));

        let mut numbers = Vec::new();
        numbers.try_push(72)?;
        numbers.try_push(108)?;
        numbers.try_push(200)?;

        Ok(RustMinimal { numbers })
    }
}

impl Drop for RustMinimal {
    fn drop(&mut self) {
        pr_info!("My numbers are {:?}\n", self.numbers);
        pr_info!("Rust minimal sample (exit)\n");
    }
}
```

Makefile:
```makefile
# SPDX-License-Identifier: GPL-2.0

KDIR ?= /download/linux-6.1.25 
# KDIR ?= /download/linux-r4l

obj-m := rust_minimal.o

default:
	$(MAKE) -C $(KDIR) M=$$PWD modules

default1:
	$(MAKE) -C $(KDIR) M=$$PWD LLVM=1 CLIPPY=1 modules
```
实际编译时发现 `LLVM=1 CLIPPY=1` 有没有都可以，`modules` 一定要有。

执行 `make` 命令后可以在当前文件夹得到 `rust_minimal.ko` 文件，将其复制到根文件系统目录，重新制作根文件系统镜像。

## 动态加载、卸载模块
启动完成后可以在根目录看到 ko 模块：
```bash
# 1. 查看模块文件
ls
bin              linuxrc          rust_minimal.ko  usr
dev              proc             sbin
etc              root             sys

# 2. 加载模块
insmod rust_minimal.ko
[    8.093904] rust_minimal: loading out-of-tree module taints kernel.
[    8.098383] rust_minimal: Rust minimal sample (init)
[    8.098497] rust_minimal: Am I built-in? false

# 3. 卸载模块
rmmod rust_minimal.ko 
[   51.045758] rust_minimal: My numbers are [72, 108, 200]
[   51.046395] rust_minimal: Rust minimal sample (exit)
```





