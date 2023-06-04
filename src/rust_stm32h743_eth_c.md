# STM32H743 以太网外设初始化分析

因为目标板与 STM32H743 使用的相同以太网 MAC，而 STM32H743 相对熟悉，并且简单，所以先从 STM32H743 这款芯片的 C 代码开始分析，了解 MAC 的初始化流程、工作原理。

参考资料：
* STM32H7-HAL 库
* STM32H7x3 参考手册 RM0433
* RT-Thread 设备驱动框架的以太网部分

## 结构体定义
有几个重要的结构体:

### ETH_InitTypeDef

该结构体用于描述 MAC 使用的外部资源信息，在初始化时参与配置 MAC 寄存器。

```c
/** 
  * @brief  ETH Init Structure definition  
  */
typedef struct
{
   
  uint8_t                     *MACAddr;         /*!< MAC Address of used Hardware: must be pointer on an array of 6 bytes */
  	
  ETH_MediaInterfaceTypeDef   MediaInterface;   /*!< Selects the MII interface or the RMII interface. */

  ETH_DMADescTypeDef          *TxDesc;          /*!< Provides the address of the first DMA Tx descriptor in the list */
  
  ETH_DMADescTypeDef          *RxDesc;          /*!< Provides the address of the first DMA Rx descriptor in the list */
  
  uint32_t                    RxBuffLen;        /*!< Provides the length of Rx buffers size */

}ETH_InitTypeDef;
```

### ETH_MACConfigTypeDef
该结构体可以认为是 MAC 配置寄存器中，所有支持功能的描述，作为 `ETH_SetMACConfig()` 的入口参数，将成员变量的值写进对应寄存器的位域中。

```c
/** 
  * @brief  ETH MAC Configuration Structure definition  
  */
typedef struct
{
  uint32_t         SourceAddrControl;           /*!< Selects the Source Address Insertion or Replacement Control.                                                           
                                                     This parameter can be a value of @ref ETH_Source_Addr_Control */                  

  FunctionalState  ChecksumOffload;             /*!< Enables or Disable the checksum checking for received packet payloads TCP, UDP or ICMP headers */    

  uint32_t         InterPacketGapVal;           /*!< Sets the minimum IPG between Packet during transmission.
                                                     This parameter can be a value of @ref ETH_Inter_Packet_Gap */   

  FunctionalState  GiantPacketSizeLimitControl; /*!< Enables or disables the Giant Packet Size Limit Control. */  

  FunctionalState  Support2KPacket;             /*!< Enables or disables the IEEE 802.3as Support for 2K length Packets */ 

  FunctionalState  CRCStripTypePacket;          /*!< Enables or disables the CRC stripping for Type packets.*/ 

  FunctionalState  AutomaticPadCRCStrip;        /*!< Enables or disables  the Automatic MAC Pad/CRC Stripping.*/ 
																													 
  FunctionalState  Watchdog;                    /*!< Enables or disables the Watchdog timer on Rx path
                                                           When enabled, the MAC allows no more then 2048 bytes to be received.
                                                           When disabled, the MAC can receive up to 16384 bytes. */  

  FunctionalState  Jabber;                      /*!< Enables or disables Jabber timer on Tx path
                                                           When enabled, the MAC allows no more then 2048 bytes to be sent.
                                                           When disabled, the MAC can send up to 16384 bytes. */

  FunctionalState  JumboPacket;                 /*!< Enables or disables receiving Jumbo Packet
                                                           When enabled, the MAC allows jumbo packets of 9,018 bytes
                                                           without reporting a giant packet error */

  uint32_t         Speed;                       /*!< Sets the Ethernet speed: 10/100 Mbps.
                                                           This parameter can be a value of @ref ETH_Speed */

  uint32_t         DuplexMode;                  /*!< Selects the MAC duplex mode: Half-Duplex or Full-Duplex mode
                                                           This parameter can be a value of @ref ETH_Duplex_Mode */

  FunctionalState  LoopbackMode;                /*!< Enables or disables the loopback mode */

  FunctionalState  CarrierSenseBeforeTransmit;  /*!< Enables or disables the Carrier Sense Before Transmission in Full Duplex Mode. */

  FunctionalState  ReceiveOwn;                  /*!< Enables or disables the Receive Own in Half Duplex mode. */  

  FunctionalState  CarrierSenseDuringTransmit;  /*!< Enables or disables the Carrier Sense During Transmission in the Half Duplex mode */

  FunctionalState  RetryTransmission;           /*!< Enables or disables the MAC retry transmission, when a collision occurs in Half Duplex mode.*/

  uint32_t         BackOffLimit;                /*!< Selects the BackOff limit value.
                                                        This parameter can be a value of @ref ETH_Back_Off_Limit */

  FunctionalState  DeferralCheck;               /*!< Enables or disables the deferral check function in Half Duplex mode. */

  uint32_t         PreambleLength;              /*!< Selects or not the Preamble Length for Transmit packets (Full Duplex mode).
                                                           This parameter can be a value of @ref ETH_Preamble_Length */ 
                                                           
  FunctionalState  UnicastSlowProtocolPacketDetect;   /*!< Enable or disables the Detection of Slow Protocol Packets with unicast address. */   

  FunctionalState  SlowProtocolDetect;          /*!< Enable or disables the Slow Protocol Detection. */   

  FunctionalState  CRCCheckingRxPackets;        /*!< Enable or disables the CRC Checking for Received Packets. */   

  uint32_t         GiantPacketSizeLimit;        /*!< Specifies the packet size that the MAC will declare it as Giant, If it's size is 
	                                                  greater than the value programmed in this field in units of bytes 
                                                          This parameter must be a number between Min_Data = 0x618 (1518 byte) and Max_Data = 0x3FFF (32 Kbyte)*/                                                          
 
  FunctionalState  ExtendedInterPacketGap;      /*!< Enable or disables the extended inter packet gap. */ 
  
  uint32_t         ExtendedInterPacketGapVal;   /*!< Sets the Extended IPG between Packet during transmission.
                                                           This parameter can be a value from 0x0 to 0xFF */ 
  
  FunctionalState  ProgrammableWatchdog;        /*!< Enable or disables the Programmable Watchdog.*/
	
  uint32_t         WatchdogTimeout;             /*!< This field is used as watchdog timeout for a received packet
                                                        This parameter can be a value of @ref ETH_Watchdog_Timeout */                                                            

   uint32_t        PauseTime;                   /*!< This field holds the value to be used in the Pause Time field in the transmit control packet. 
                                                   This parameter must be a number between Min_Data = 0x0 and Max_Data = 0xFFFF */

  FunctionalState  ZeroQuantaPause;             /*!< Enable or disables the automatic generation of Zero Quanta Pause Control packets.*/  

  uint32_t         PauseLowThreshold;           /*!< This field configures the threshold of the PAUSE to be checked for automatic retransmission of PAUSE Packet.
                                                   This parameter can be a value of @ref ETH_Pause_Low_Threshold */

  FunctionalState  TransmitFlowControl;         /*!< Enables or disables the MAC to transmit Pause packets in Full Duplex mode
                                                   or the MAC back pressure operation in Half Duplex mode */

  FunctionalState  UnicastPausePacketDetect;    /*!< Enables or disables the MAC to detect Pause packets with unicast address of the station */
  
  FunctionalState  ReceiveFlowControl;          /*!< Enables or disables the MAC to decodes the received Pause packet  
                                                  and disables its transmitter for a specified (Pause) time */
																													                                                        	 
  uint32_t         TransmitQueueMode;           /*!< Specifies the Transmit Queue operating mode.
                                                      This parameter can be a value of @ref ETH_Transmit_Mode */ 

  uint32_t         ReceiveQueueMode;            /*!< Specifies the Receive Queue operating mode.
                                                             This parameter can be a value of @ref ETH_Receive_Mode */

  FunctionalState  DropTCPIPChecksumErrorPacket; /*!< Enables or disables Dropping of TCPIP Checksum Error Packets. */ 

  FunctionalState  ForwardRxErrorPacket;        /*!< Enables or disables  forwarding Error Packets. */ 

  FunctionalState  ForwardRxUndersizedGoodPacket;  /*!< Enables or disables  forwarding Undersized Good Packets.*/ 
} ETH_MACConfigTypeDef;
```

### ETH_TypeDef

片内以太网外设的寄存器映射，其中包含以下寄存器映射：
* DMA 寄存器
* MTL 寄存器
* MAC 寄存器

```c
/**
  * @brief Ethernet MAC
  */
typedef struct
{
  __IO uint32_t MACCR;
  __IO uint32_t MACECR;
  __IO uint32_t MACPFR;
  __IO uint32_t MACWTR;
  __IO uint32_t MACHT0R;
  __IO uint32_t MACHT1R;
  uint32_t      RESERVED1[14];
  __IO uint32_t MACVTR;
  uint32_t      RESERVED2;
  __IO uint32_t MACVHTR;
  uint32_t      RESERVED3;
  __IO uint32_t MACVIR;
  __IO uint32_t MACIVIR;
  uint32_t      RESERVED4[2];
  __IO uint32_t MACTFCR;
  uint32_t      RESERVED5[7];
  __IO uint32_t MACRFCR;
  uint32_t      RESERVED6[7];
  __IO uint32_t MACISR;
  __IO uint32_t MACIER;
  __IO uint32_t MACRXTXSR;
  uint32_t      RESERVED7;
  __IO uint32_t MACPCSR;
  __IO uint32_t MACRWKPFR;
  uint32_t      RESERVED8[2];
  __IO uint32_t MACLCSR;
  __IO uint32_t MACLTCR;
  __IO uint32_t MACLETR;
  __IO uint32_t MAC1USTCR;
  uint32_t      RESERVED9[12];
  __IO uint32_t MACVR;
  __IO uint32_t MACDR;
  uint32_t      RESERVED10;
  __IO uint32_t MACHWF0R;
  __IO uint32_t MACHWF1R;
  __IO uint32_t MACHWF2R;
  uint32_t      RESERVED11[54];
  __IO uint32_t MACMDIOAR;
  __IO uint32_t MACMDIODR;
  uint32_t      RESERVED12[2];
  __IO uint32_t MACARPAR;
  uint32_t      RESERVED13[59];
  __IO uint32_t MACA0HR;
  __IO uint32_t MACA0LR;
  __IO uint32_t MACA1HR;
  __IO uint32_t MACA1LR;
  __IO uint32_t MACA2HR;
  __IO uint32_t MACA2LR;
  __IO uint32_t MACA3HR;
  __IO uint32_t MACA3LR;
  uint32_t      RESERVED14[248];
  __IO uint32_t MMCCR;
  __IO uint32_t MMCRIR;
  __IO uint32_t MMCTIR;
  __IO uint32_t MMCRIMR;
  __IO uint32_t MMCTIMR;
  uint32_t      RESERVED15[14];
  __IO uint32_t MMCTSCGPR;
  __IO uint32_t MMCTMCGPR;
  uint32_t      RESERVED16[5];
  __IO uint32_t MMCTPCGR;
  uint32_t      RESERVED17[10];
  __IO uint32_t MMCRCRCEPR;
  __IO uint32_t MMCRAEPR;
  uint32_t      RESERVED18[10];
  __IO uint32_t MMCRUPGR;
  uint32_t      RESERVED19[9];
  __IO uint32_t MMCTLPIMSTR;
  __IO uint32_t MMCTLPITCR;
  __IO uint32_t MMCRLPIMSTR;
  __IO uint32_t MMCRLPITCR;
  uint32_t      RESERVED20[65];
  __IO uint32_t MACL3L4C0R;
  __IO uint32_t MACL4A0R;
  uint32_t      RESERVED21[2];
  __IO uint32_t MACL3A0R0R;
  __IO uint32_t MACL3A1R0R;
  __IO uint32_t MACL3A2R0R;
  __IO uint32_t MACL3A3R0R;
  uint32_t      RESERVED22[4];
  __IO uint32_t MACL3L4C1R;
  __IO uint32_t MACL4A1R;
  uint32_t      RESERVED23[2];
  __IO uint32_t MACL3A0R1R;
  __IO uint32_t MACL3A1R1R;
  __IO uint32_t MACL3A2R1R;
  __IO uint32_t MACL3A3R1R;
  uint32_t      RESERVED24[108];
  __IO uint32_t MACTSCR;
  __IO uint32_t MACSSIR;
  __IO uint32_t MACSTSR;
  __IO uint32_t MACSTNR;
  __IO uint32_t MACSTSUR;
  __IO uint32_t MACSTNUR;
  __IO uint32_t MACTSAR;
  uint32_t      RESERVED25;
  __IO uint32_t MACTSSR;
  uint32_t      RESERVED26[3];
  __IO uint32_t MACTTSSNR;
  __IO uint32_t MACTTSSSR;
  uint32_t      RESERVED27[2];
  __IO uint32_t MACACR;
  uint32_t      RESERVED28;
  __IO uint32_t MACATSNR;
  __IO uint32_t MACATSSR;
  __IO uint32_t MACTSIACR;
  __IO uint32_t MACTSEACR;
  __IO uint32_t MACTSICNR;
  __IO uint32_t MACTSECNR;
  uint32_t      RESERVED29[4];
  __IO uint32_t MACPPSCR;
  uint32_t      RESERVED30[3];
  __IO uint32_t MACPPSTTSR;
  __IO uint32_t MACPPSTTNR;
  __IO uint32_t MACPPSIR;
  __IO uint32_t MACPPSWR;
  uint32_t      RESERVED31[12];
  __IO uint32_t MACPOCR;
  __IO uint32_t MACSPI0R;
  __IO uint32_t MACSPI1R;
  __IO uint32_t MACSPI2R;
  __IO uint32_t MACLMIR;
  uint32_t      RESERVED32[11];
  __IO uint32_t MTLOMR;
  uint32_t      RESERVED33[7];
  __IO uint32_t MTLISR;
  uint32_t      RESERVED34[55];
  __IO uint32_t MTLTQOMR;
  __IO uint32_t MTLTQUR;
  __IO uint32_t MTLTQDR;
  uint32_t      RESERVED35[8];
  __IO uint32_t MTLQICSR;
  __IO uint32_t MTLRQOMR;
  __IO uint32_t MTLRQMPOCR;
  __IO uint32_t MTLRQDR;
  uint32_t      RESERVED36[177];
  __IO uint32_t DMAMR;
  __IO uint32_t DMASBMR;
  __IO uint32_t DMAISR;
  __IO uint32_t DMADSR;
  uint32_t      RESERVED37[60];
  __IO uint32_t DMACCR;
  __IO uint32_t DMACTCR;
  __IO uint32_t DMACRCR;
  uint32_t      RESERVED38[2];
  __IO uint32_t DMACTDLAR;
  uint32_t      RESERVED39;
  __IO uint32_t DMACRDLAR;
  __IO uint32_t DMACTDTPR;
  uint32_t      RESERVED40;
  __IO uint32_t DMACRDTPR;
  __IO uint32_t DMACTDRLR;
  __IO uint32_t DMACRDRLR;
  __IO uint32_t DMACIER;
  __IO uint32_t DMACRIWTR;
__IO uint32_t DMACSFCSR;
  uint32_t      RESERVED41;
  __IO uint32_t DMACCATDR;
  uint32_t      RESERVED42;
  __IO uint32_t DMACCARDR;
  uint32_t      RESERVED43;
  __IO uint32_t DMACCATBR;
  uint32_t      RESERVED44;
  __IO uint32_t DMACCARBR;
  __IO uint32_t DMACSR;
uint32_t      RESERVED45[2];
__IO uint32_t DMACMFCR;
}ETH_TypeDef;
```

## STM32-HAL 中 eth 初始化
STM32 有一套完整的硬件抽象层（HAL）库，其目的除了抽象硬件，方便用户编写业务代码，更主要的是统一自家芯片的寄存器配置 API 、编程模型，是其初始化代码生成器 STM32CubeMX 所使用的库。

HAL 库中 eth 外设初始化代码如下：
```c
/**
  * @brief  Initialize the Ethernet peripheral registers.
  * @param  heth: pointer to a ETH_HandleTypeDef structure that contains
  *         the configuration information for ETHERNET module
  * @retval HAL status
  */
HAL_StatusTypeDef HAL_ETH_Init(ETH_HandleTypeDef *heth)
{
  uint32_t tickstart;

  if(heth == NULL)
  {
    return HAL_ERROR;
  }

#if (USE_HAL_ETH_REGISTER_CALLBACKS == 1)

  if(heth->gState == HAL_ETH_STATE_RESET)
  {
    /* Allocate lock resource and initialize it */
    heth->Lock = HAL_UNLOCKED;

    ETH_InitCallbacksToDefault(heth);

    if(heth->MspInitCallback == NULL)
    {
      heth->MspInitCallback = HAL_ETH_MspInit;
    }

    /* Init the low level hardware */
    heth->MspInitCallback(heth);
  }

#else

  /* Check the ETH peripheral state */
  if(heth->gState == HAL_ETH_STATE_RESET)
  {
    /* Init the low level hardware : GPIO, CLOCK, NVIC. */
    HAL_ETH_MspInit(heth);
  }
#endif /* (USE_HAL_ETH_REGISTER_CALLBACKS) */

  heth->gState = HAL_ETH_STATE_BUSY;

  __HAL_RCC_SYSCFG_CLK_ENABLE();

  if(heth->Init.MediaInterface == HAL_ETH_MII_MODE)
  {
    HAL_SYSCFG_ETHInterfaceSelect(SYSCFG_ETH_MII);
  }
  else
  {
    HAL_SYSCFG_ETHInterfaceSelect(SYSCFG_ETH_RMII);
  }

  /* Ethernet Software reset */
  /* Set the SWR bit: resets all MAC subsystem internal registers and logic */
  /* After reset all the registers holds their respective reset values */
  SET_BIT(heth->Instance->DMAMR, ETH_DMAMR_SWR);

  /* Get tick */
  tickstart = HAL_GetTick();

  /* Wait for software reset */
  while (READ_BIT(heth->Instance->DMAMR, ETH_DMAMR_SWR) > 0U)
  {
    if(((HAL_GetTick() - tickstart ) > ETH_SWRESET_TIMEOUT))
    {
      /* Set Error Code */
      heth->ErrorCode = HAL_ETH_ERROR_TIMEOUT;
      /* Set State as Error */
      heth->gState = HAL_ETH_STATE_ERROR;
      /* Return Error */
      return HAL_ERROR;
    }
  }

  /*------------------ MDIO CSR Clock Range Configuration --------------------*/
  ETH_MAC_MDIO_ClkConfig(heth);

  /*------------------ MAC LPI 1US Tic Counter Configuration --------------------*/
  WRITE_REG(heth->Instance->MAC1USTCR, (((uint32_t)HAL_RCC_GetHCLKFreq() / ETH_MAC_US_TICK) - 1U));

  /*------------------ MAC, MTL and DMA default Configuration ----------------*/
  ETH_MACDMAConfig(heth);

  /* SET DSL to 64 bit */
  MODIFY_REG(heth->Instance->DMACCR, ETH_DMACCR_DSL, ETH_DMACCR_DSL_64BIT);

  /* Set Receive Buffers Length (must be a multiple of 4) */
  if ((heth->Init.RxBuffLen % 0x4U) != 0x0U)
  {
    /* Set Error Code */
    heth->ErrorCode = HAL_ETH_ERROR_PARAM;
    /* Set State as Error */
    heth->gState = HAL_ETH_STATE_ERROR;
    /* Return Error */
    return HAL_ERROR;
  }
  else
  {
    MODIFY_REG(heth->Instance->DMACRCR, ETH_DMACRCR_RBSZ, ((heth->Init.RxBuffLen) << 1));
  }

  /*------------------ DMA Tx Descriptors Configuration ----------------------*/
  ETH_DMATxDescListInit(heth);

  /*------------------ DMA Rx Descriptors Configuration ----------------------*/
  ETH_DMARxDescListInit(heth);

  /*--------------------- ETHERNET MAC Address Configuration ------------------*/
  /* Set MAC addr bits 32 to 47 */
  heth->Instance->MACA0HR = (((uint32_t)(heth->Init.MACAddr[5]) << 8) | (uint32_t)heth->Init.MACAddr[4]);
  /* Set MAC addr bits 0 to 31 */
  heth->Instance->MACA0LR = (((uint32_t)(heth->Init.MACAddr[3]) << 24) | ((uint32_t)(heth->Init.MACAddr[2]) << 16) |
                             ((uint32_t)(heth->Init.MACAddr[1]) << 8) | (uint32_t)heth->Init.MACAddr[0]);

  heth->ErrorCode = HAL_ETH_ERROR_NONE;
  heth->gState = HAL_ETH_STATE_READY;
  heth->RxState = HAL_ETH_STATE_READY;

  return HAL_OK;
}
```
初始化流程：
1. 板级初始化：硬件、时钟等配置
2. 与 PHY 芯片的连接方式配置（MII/RMII），寄存器 `SYSCFG::PMCR::EPIS`
3. 软复位 eth 外设，并等待复位完成，寄存器 `ETH::DMAMR::SWR`
4. 配置 MDIO 时钟范围，寄存器 `ETH::MACMDIOAR::CR`
5. 配置 1 us 计数值 ，寄存器 `ETH::MAC1USTCR`
6. 配置 MAC、MTL、DMA 寄存器的默认值，其中
    1. MAC、MTL 默认配置，寄存器：`ETH::MACCR`, `ETH::MACECR`, `ETH::MACWTR`, `ETH::MACTFCR`, `ETH::MACRFCR`, `ETH::MTLTQOMR`, `ETH::MTLRQOMR`
    2. DMA 默认配置，寄存器 `ETH::DMAMR`, `ETH::DMASBMR`, `ETH::DMACCR`, `ETH::DMACTCR`, `ETH::DMACRCR`
7. 配置描述符跳过长度，寄存器 `ETH::DMACCR::DSL`
8. 配置接收缓冲区长度，寄存器 `ETH::DMACRCR`
9. 配置 Tx 描述符
10. 配置 Rx 描述符
11. 配置 MAC 地址，寄存器 `ETH::MACA0HR`, `ETH::MACA0LR`

其中： <br>
第 2 步，具体是配置的 SYSCFG::PMCR::EPIS，系统配置控制器（SYSCFG）不做详细介绍，这里用到了外设模式配置寄存器（PMCR）的以太网 PHY 接口选择（EPIS）位域，该位域共 3 bit，b000 为 MII，b100 为 RMII，其余保留。



## RTT 中 eth 初始化

RTT 有一套自己的设备驱动框架，与 Linux 相似，其中 eth 的初始化函数如下：
```c
extern void phy_reset(void);
/* EMAC initialization function */
static rt_err_t rt_stm32_eth_init(rt_device_t dev) {
    ETH_MACConfigTypeDef MACConf;
    uint32_t regvalue = 0;
    uint8_t status = RT_EOK;

    __HAL_RCC_D2SRAM3_CLK_ENABLE();

    phy_reset();

    /* ETHERNET Configuration */
    EthHandle.Instance = ETH;
    EthHandle.Init.MACAddr = (rt_uint8_t *)&stm32_eth_device.dev_addr[0];
    EthHandle.Init.MediaInterface = HAL_ETH_RMII_MODE;
    EthHandle.Init.TxDesc = DMATxDscrTab;
    EthHandle.Init.RxDesc = DMARxDscrTab;
    EthHandle.Init.RxBuffLen = ETH_MAX_PACKET_SIZE;

    // SCB_InvalidateDCache();

    HAL_ETH_DeInit(&EthHandle);

    /* configure ethernet peripheral (GPIOs, clocks, MAC, DMA) */
    if (HAL_ETH_Init(&EthHandle) != HAL_OK) {
        LOG_E("eth hardware init failed");
    } else {
        LOG_D("eth hardware init success");
    }

    rt_memset(&TxConfig, 0, sizeof(ETH_TxPacketConfig));
    TxConfig.Attributes = ETH_TX_PACKETS_FEATURES_CSUM | ETH_TX_PACKETS_FEATURES_CRCPAD;
    TxConfig.ChecksumCtrl = ETH_CHECKSUM_IPHDR_PAYLOAD_INSERT_PHDR_CALC;
    TxConfig.CRCPadCtrl = ETH_CRC_PAD_INSERT;

    for (int idx = 0; idx < ETH_RX_DESC_CNT; idx++) {
        HAL_ETH_DescAssignMemory(&EthHandle, idx, &Rx_Buff[idx][0], NULL);
    }

    HAL_ETH_SetMDIOClockRange(&EthHandle);

    for (int i = 0; i <= PHY_ADDR; i++) {
        if (HAL_ETH_ReadPHYRegister(&EthHandle, i, PHY_SPECIAL_MODES_REG, &regvalue) != HAL_OK) {
            status = RT_ERROR;
            /* Can't read from this device address continue with next address */
            continue;
        }

        if ((regvalue & PHY_BASIC_STATUS_REG) == i) {
            PHY_ADDR = i;
            status = RT_EOK;
            LOG_D("Found a phy, address:0x%02X", PHY_ADDR);
            break;
        }
    }

    if (HAL_ETH_WritePHYRegister(&EthHandle, PHY_ADDR, PHY_BASIC_CONTROL_REG, PHY_RESET_MASK) ==
        HAL_OK) {
        HAL_ETH_ReadPHYRegister(&EthHandle, PHY_ADDR, PHY_SPECIAL_MODES_REG, &regvalue);

        uint32_t tickstart = rt_tick_get();

        /* wait until software reset is done or timeout occured  */
        while (regvalue & PHY_RESET_MASK) {
            if ((rt_tick_get() - tickstart) <= 500) {
                if (HAL_ETH_ReadPHYRegister(&EthHandle, PHY_ADDR, PHY_BASIC_CONTROL_REG,
                                            &regvalue) != HAL_OK) {
                    status = RT_ERROR;
                    break;
                }
            } else {
                status = RT_ETIMEOUT;
            }
        }
    }

    rt_thread_delay(2000);

    if (HAL_ETH_ReadPHYRegister(&EthHandle, PHY_ADDR, PHY_BASIC_CONTROL_REG, &regvalue) == HAL_OK) {
        regvalue |= PHY_AUTO_NEGOTIATION_MASK;
        HAL_ETH_WritePHYRegister(&EthHandle, PHY_ADDR, PHY_BASIC_CONTROL_REG, regvalue);

        eth_device_linkchange(&stm32_eth_device.parent, RT_TRUE);
        HAL_ETH_GetMACConfig(&EthHandle, &MACConf);
        MACConf.DuplexMode = ETH_FULLDUPLEX_MODE;
        MACConf.Speed = ETH_SPEED_100M;
        HAL_ETH_SetMACConfig(&EthHandle, &MACConf);

        HAL_ETH_Start_IT(&EthHandle);
    } else {
        status = RT_ERROR;
    }

    return status;
}
```

初始化流程：
1. PHY 芯片的硬件复位
2. 定义以太网用到的资源，即 `ETH_InitTypeDef` 结构体赋值
3. 初始化 eth 外设，即[上一节](#stm32-hal-中-eth-初始化)介绍的 `HAL_ETH_Init()` 函数
4. 配置发送包的部分值，这里的值在后续使用中不再变动，后续使用是只需要修改 `Length` 和 `TxBuffer` 两个当前发送内容相关的变量
5. 轮询 0～0x1F 地址，读一个特殊寄存器，如果能读到数据，则保存当前地址，并退出轮询（TODO: 根据 PHY 型号，查看该寄存器含义）
6. 软复位 PHY 芯片，通过写通用控制寄存器中的复位位，并等待复位完成
7. 开启自动协商，写通用控制寄存器中的自动协商位，等待协商完成后将协商结果，即速率和双工模式写回 eth 外设。这里的代码略有不同，因为使用的 PHY 比较特殊，自带双网口交换机，与单片机连接的端口是中是100M全双工。
