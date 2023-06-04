# 寄存器对比

BST A1000 GMAC Register Description

[STM32H743 RM0433.pdf](./resource/rm0433-stm32h742-stm32h743753-and-stm32h750.pdf)

## GMAC Registers

### Reg 0 0x0000
*0x0800E003*

**STM32:** <br>
工作模式配置寄存器 ETH_MACCR
* bit27 - 校验和减荷，会使能 IPv4 报头校验和检查以及 IPv4 或 IPv6 TCP、UDP 或 ICMP 有效负载 
校验和检查

* bit15 - 保留，必须为复位值(0)
* bit14 - MAC速度，0：10Mbps，1：100Mbps
* bit13 - 双工模式，1：全双工
* bit1 - 发送使能
* bit0 - 接收使能

**A1000:** <br>
EQOS_MAC_R_MAC_CONFIGURATION
* bit15 - Port Select, 与 bit14 联合指示确切速度。在 10/100 Mbps-only （固定1），或 1000Mbps-only（固定0）模式下为只读，在 10/100/1000Mbps 配置下，为读写，0：1000～2500Mbps 选项，1：10～100Mbps 选项
* bit14 - Speed，bit15 为 1 时，0：10Mbps，1：100Mbps，bit15 为 0 时，0：1Gbps，1：2.5Gbps

### Reg1 0x0004
*0x00000000*

**STM32:** <br>


**A1000:** <br>

### Reg2 0x0008
*0x00010404*

**STM32:** <br>
数据包过滤控制寄存器 ETH_MACPFR
* bit16 - VLAN 标记过滤器时能，MAC 将对其与 VLAN 标记不匹配的带 VLAN 标记的数据包
* bit10 - Hash 或 Perfect 过滤器，如果数据包与完美过滤或散列过滤（通过 HMC 或 HUC 位设置）匹配，则地址 
过滤器会通过此数据包
* bit2 - Hash 多播，MAC 根据 Hash 表对接收到的多播数据包执行目标地址过滤

**A1000:** <br>
*同上*

### Reg3 0x000C
*0x00000000*

**STM32:** <br>


**A1000:** <br>

### Reg4 0x0010
*0x00000002*

**STM32:** <br>
散列表 0 寄存器 ETH_MACHT0R
* 64位散列表的低32位 即 [31:0]

**A1000:** <br>
*同上*

### Reg5 0x0014
*0x04000001*

**STM32:** <br>
散列表 1 寄存器 ETH_MACHT1R
* 64位散列表的高32位 即 [63:32]

**A1000:** <br>
*同上*

### Reg6 0x0018
*0x00000000*

**STM32:** <br>


**A1000:** <br>

### Reg7 0x001C
*0x00000000*

**STM32:** <br>


**A1000:** <br>

### Reg8 0x0020
*0x00000000*

**STM32:** <br>


**A1000:** <br>

### Reg9 0x0024
*0x00000000*

**STM32:** <br>


**A1000:** <br>

### Reg10 0x0028
*0x00000000*

**STM32:** <br>


**A1000:** <br>

### Reg11 0x002C
*0x00000000*

**STM32:** <br>


**A1000:** <br>

### Reg12 0x0030
*0x00000000*

**STM32:** <br>


**A1000:** <br>

### Reg13 0x0034
*0x00000000*

**STM32:** <br>


**A1000:** <br>

### Reg14 0x0038
*0x00000000*

**STM32:** <br>


**A1000:** <br>

### Reg15 0x003C
*0x00000000*

**STM32:** <br>


**A1000:** <br>

### Reg16 0x0040
*0x00000000*

**STM32:** <br>


**A1000:** <br>

### Reg17 0x0044
*0x00000000*

**STM32:** <br>


**A1000:** <br>

### Reg18 0x0048
*0x00000000*

**STM32:** <br>


**A1000:** <br>

### Reg19 0x004C
*0x00000000*

**STM32:** <br>


**A1000:** <br>

### Reg20 0x0050
*0x0201001C*

**STM32:** <br>
VLAN 标记寄存器 ETH_MACVTR
* bit25 - 散列表匹配使能
* bit16 - 12 位 VLAN 标记比较使能
* bit[15:0] - 用于接收数据包的 VLAN 标记标识符

**A1000:** <br>
* bit25 - 散列表匹配使能
* bit16 - 12 位 VLAN 标记比较使能
* bit[4:2] - 偏移，应用程序尝试访问的 MAC VLAN 标记过滤器寄存器的地址偏移量

### Reg21 0x0054
*0x00000000*

**STM32:** <br>


**A1000:** <br>

### Reg22 0x0058
*0x00000001*

**STM32:** <br>
VLAN 散列表寄存器 ETH_MACVHTR
* bit[15:0] - 16位 VLAN 散列表

**A1000:** <br>
*同上*

### Reg23 0x005C
*0x00000000*

**STM32:** <br>


**A1000:** <br>

### Reg24 0x0060
*0x00000000*

**STM32:** <br>


**A1000:** <br>

### Reg25 0x0064
*0x00000000*

**STM32:** <br>


**A1000:** <br>

### Reg26 0x0068
*0x00000000*

**STM32:** <br>


**A1000:** <br>

### Reg27 0x006C
*0x00000000*

**STM32:** <br>


**A1000:** <br>

### Reg28 0x0070
*0x00000000*

**STM32:** <br>


**A1000:** <br>

### Reg29 0x0074
*0x00000000*

**STM32:** <br>


**A1000:** <br>

### Reg30 0x0078
*0x00000000*

**STM32:** <br>


**A1000:** <br>

### Reg31 0x007C
*0x00000000*

**STM32:** <br>


**A1000:** <br>

### Reg32 0x0080
*0x00000000*

**STM32:** <br>


**A1000:** <br>

### Reg33 0x0084
*0x00000000*

**STM32:** <br>


**A1000:** <br>

### Reg34 0x0088
*0x00000000*

**STM32:** <br>


**A1000:** <br>

### Reg35 0x008C
*0x00000000*

**STM32:** <br>


**A1000:** <br>

### Reg36 0x0090
*0x00000000*

**STM32:** <br>


**A1000:** <br>

### Reg37 0x0094
*0x00000000*

**STM32:** <br>


**A1000:** <br>

### Reg38 0x0098
*0x00000000*

**STM32:** <br>


**A1000:** <br>

### Reg39 0x009C
*0x00000000*

**STM32:** <br>


**A1000:** <br>

### Reg40 0x00A0
*0x000000AA*

**STM32:** <br>
*保留*

**A1000:** <br>
EQOS_MAC_R_MAC_RXQ_CTRL0
* bit[7:6] - 接收队列3使能，2：使能 DCB/Generic 队列
* bit[5:4] - 接收队列2使能，2：使能 DCB/Generic 队列
* bit[3:2] - 接收队列1使能，2：使能 DCB/Generic 队列
* bit[1:0] - 接收队列0使能，2：使能 DCB/Generic 队列

### Reg41 0x00A4
*0x00000000*

**STM32:** <br>


**A1000:** <br>

### Reg42 0x00A8
*0x03020100*

**STM32:** <br>
*保留*

**A1000:** <br>
EQOS_MAC_R_MAC_RXQ_CTRL2
* bit[31:24] - 接收队列3的优先级，3
* bit[23:16] - 接收队列2的优先级，2
* bit[15:0] - 接收队列1的优先级，1
* bit[7:0] - 接收队列0的优先级，0


### Reg43 0x00AC
*0x00000000*

**STM32:** <br>


**A1000:** <br>

### Reg44 0x00B0
*0x00040000*

**STM32:** <br>
中断状态寄存器 ETH_MACISR
* bit[31:16] - 保留

**A1000:** <br>
EQOS_MAC_R_MAC_INTERRUPT_STATUS
* bit18 - MDIO Interrupt Status，MDIO 完成操作中断指示位

### Reg45 0x00B4
*0x00000000*

**STM32:** <br>


**A1000:** <br>

### Reg46 0x00B8
*0x00000000*

**STM32:** <br>


**A1000:** <br>

### Reg47 0x00BC
*0x00000000*

**STM32:** <br>


**A1000:** <br>

### Reg48 0x00C0
*0x00000000*

**STM32:** <br>


**A1000:** <br>

### Reg49 0x00C4
*0x00000000*

**STM32:** <br>


**A1000:** <br>

### Reg50 0x00C8
*0x00000000*

**STM32:** <br>


**A1000:** <br>

### Reg51 0x00CC
*0x00000000*

**STM32:** <br>


**A1000:** <br>

### Reg52 0x00D0
*0x00000000*

**STM32:** <br>


**A1000:** <br>

### Reg53 0x00D4
*0x03E80000*

**STM32:** <br>
LPI 定时器控制寄存器 ETH_MACLTCR
* bit[25:16] - LPI LS 定时器，在可以将 LPI 模式发送到 PHY 之前，来自 PHY 的链路状态 (OKAY) 应持续的最短 
时间（以毫秒为单位），1000

**A1000:** <br>
*同上*

### Reg54 0x00D8
*0x00000000*

**STM32:** <br>


**A1000:** <br>

## DMA Registers

> 未完成，寄存器对应关系不对
> 
> 直接查看 [寄存器说明](./rust_register_info.md)

### Reg0 0x1000
*0x00010000*

**STM32:** <br>
DMA 模式寄存器 ETH_DMAMR
* bit[17:16] - 以太网外设的中断模式，<br>0：检测到传送完成事件时，TI/RI 状态信号会被置 1。软件驱动程序向这些位写入 1 时，这些位会被清零。如果还在 ETH_DMACIER 寄存器中使能了相应的中断，则中断信号会被置为有效；<br>1：如上所述，TI/RI 置 1。不过，对于任一 RI/TI 事件，都不会触发中断；<br>2：RI/TI 状态位会在检测到传送完成事件时置 1，并且会在软件驱动程序通过写入 1 来将其清零时复位。不过，如果在软件将其清除（处理）之前检测到另一个传送完成事件，则会自动再次将这些状态位置 1。但是，并非基于 TI/RI 生成中断。

**A1000:** <br>
*同上*

### Reg1 0x1004
*0x00080011*

**STM32:** <br>
系统总线模式寄存器 ETH_DMASBMR
* bit19 - 保留
* bit1 - 保留
* bit0 - 固定突发长度，1：AHB 主设备将发起特定长度的突发传输（INCRx 或 SINGLE）

**A1000:** <br>
* bit[19:16] - AXI 最大读取未完成请求限制，8
* bit1 - AXI 突发长度为4
* bit0 - 固定突发长度

### Reg2 0x1008
*0x00080C01*

**STM32:** <br>
中断状态寄存器 ETH_DMAISR
* bit19 - 保留
* bit11 - 保留
* bit10 - 保留
* bit0 - DMA 通道中断状态，此位指示 DMA 通道中发生的中断事件。要将该位复位为 0，软件必须读取 DMA 通道中的相应寄存器，以获取产生中断的具体原因，然后将中断源清除。

**A1000:** <br>
* bit0 - DMA Ch0 中断状态，清除条件同上。

### Reg3 0x100C
*0x00000000*

**STM32:** <br>
ETH_DMADSR

**A1000:** <br>

### Reg4 0x1100
*0x00000000*

**STM32:** <br>
ETH_DMACCR

**A1000:** <br>

### Reg5 0x1104
*0x83194000*

**STM32:** <br>
ETH_DMACTxCR

**A1000:** <br>

### Reg6 0x1108
*0x00000000*

**STM32:** <br>
ETH_DMACRxCR

**A1000:** <br>

### Reg7 0x110C
*0x83186000*

**STM32:** <br>
*保留*

**A1000:** <br>

### Reg8 0x1100
*0x83194000*

**STM32:** <br>
*保留*

**A1000:** <br>

### Reg9 0x1114
*0x00000000*

**STM32:** <br>


**A1000:** <br>

### Reg10 0x1028
*0x83188000*

**STM32:** <br>
ETH_DMACTxDLAR

**A1000:** <br>

### Reg11 0x111C
*0x000001FF*

**STM32:** <br>
ETH_DMACRxDLAR

**A1000:** <br>

### Reg12 0x1120
*0x000001FF*

**STM32:** <br>
ETH_DMACTxDTPR

**A1000:** <br>

### Reg13 0x1124
*0x0000D041*

**STM32:** <br>
*保留*

**A1000:** <br>

### Reg14 0x1128
*0x00000000*

**STM32:** <br>
ETH_DMACRxDTPR

**A1000:** <br>

### Reg15 0x112C
*0x000007C0*

**STM32:** <br>
ETH_DMACTxDRLR

**A1000:** <br>

### Reg16 0x1130
*0x00000000*

**STM32:** <br>
ETH_DMACRxDRLR

**A1000:** <br>

### Reg17 0x1134
*0x83194000*

**STM32:** <br>
ETH_DMACIER

**A1000:** <br>

### Reg18 0x1138
*0x00000000*

**STM32:** <br>
ETH_DMACRxIWTR

**A1000:** <br>

### Reg19 0x1140
*0x83186260*

**STM32:** <br>
*保留*

**A1000:** <br>

### Reg20 0x1050
*0x00000000*

**STM32:** <br>


**A1000:** <br>

### Reg21 0x1054
*0x00000000*

**STM32:** <br>
ETH_DMACCATxBR

**A1000:** <br>

### Reg22 0x116C
*0x00000000*

**STM32:** <br>
ETH_DMACMFCR

**A1000:** <br>

