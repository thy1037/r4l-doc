# LLVM 修复记录

处理 rust-for-linux-6.1 [issues 2](https://github.com/os2edu/rust-for-linux-6.1/issues/2)，即解决在原工程加入 `LLVM=1` 编译选项后出现的编译时错误、警告。

以下是过程中出现的错误、警告，及其对应的修改方案：

## Kconfig 中的重定义警告
**编译提示:**

`drivers/pinctrl/bst/Kconfig:3:warning: ignoring type redefinition of 'PINCTRL_MSM' from 'tristate' to 'bool'`

**出现原因：**

重复以不同类型定义了 `PINCTRL_MSM`， 在 `drivers/pinctrl/bst/Kconfig:3` 中定义了 `bool` 类型 ，而在 `drivers/pinctrl/qcom/Kconfig:4` 中定义了 `tristate` 类型。

**解决方案：**

修改 `drivers/pinctrl/bst/Kconfig:3` 中的定义为 `tristate`，并添加默认值为 `default y`

**git commit**

## 非易失性（non-volatile）空指针的赋值错误
**编译提示:**

`../drivers/gpu/arm/mali/linux/mali_osk_misc.c:54:2: error: indirection of non-volatile null pointer will be deleted, not trap [-Werror,-Wnull-dereference]`

**出现原因：**

本义是人为制造一个运行时错误，将 0 地址强转为整形指针并赋值：`*(int *)0 = 0;`，llvm 有更严格的编译检查，认为对空地址的赋值需要是 `volatile` 的。

**解决方案：**

`*(volatile int *)0 = (volatile int)0;`

**git commit**

