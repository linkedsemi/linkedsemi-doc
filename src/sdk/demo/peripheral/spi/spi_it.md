# SPI_IT 使用示例

例程路径: ls_sdk\examples\peripheral\spi_i2s\spi_it_master
          ls_sdk\examples\peripheral\spi_i2s\spi_it_slave

## 一、程序基本配置及说明：

spi_it示例程序演示了使用中断的方式实现spi master和spi slave之间的数据传输，例程以100ms的间隔传输10组数据

程序开始时先进行系统初始化和spi初始化：

```c
    /* system init app     */
    sys_init_none();
    /* init spi and GPIO   */
    spi_init();
```

spi IO端口设置：

        /* Configure the GPIO AF */
        /* CLK-------------PB12 */	
        /* CS--------------PB13 */	
        /* MOSI------------PB14 */	
        /* MISO------------PB15 */
```C
    /* master device */
    spi2_master_cs_init(SPI_CS_PIN);
    pinmux_spi2_master_clk_init(SPI_CLK_PIN,SPI_POLARITY_LOW);  // The idle state of clock must correspond to the polarity
    pinmux_spi2_master_mosi_init(SPI_MOSI_PIN); 
    pinmux_spi2_master_miso_init(SPI_MISO_PIN);
 
    /* slave device */
    pinmux_spi2_slave_clk_init(SPI_CLK_PIN);
    pinmux_spi2_slave_nss_init(SPI_CS_PIN);
    pinmux_spi2_slave_mosi_init(SPI_MOSI_PIN);
    pinmux_spi2_slave_miso_init(SPI_MISO_PIN);
```

## 二、操作步骤及结果：

### 2.1 操作步骤

**说明**：

SPI初始化结构体配置说明：

```C
    /* Set the SPI parameters */
    SpiHandle.Instance               = SPI2;   						        /*选择SPI Instance */
    SpiHandle.Init.BaudRatePrescaler = SPI_BAUDRATEPRESCALER_64;		    /*设置时钟分频因子，fpclk/分频数=fSCK */
    SpiHandle.Init.CLKPhase          = SPI_PHASE_1EDGE;				        /*设置时钟相位，可选奇/偶数边沿采样 */
    SpiHandle.Init.CLKPolarity       = SPI_POLARITY_LOW;				    /*设置时钟极性CPOL，可选高/低电平*/
    SpiHandle.Init.DataSize          = SPI_DATASIZE_8BIT;				    /*设置SPI的数据帧长度，可选8/16位 */
    SpiHandle.Init.FirstBit          = SPI_FIRSTBIT_MSB;				    /*设置MSB/LSB先行 */
    SpiHandle.Init.TIMode            = SPI_TIMODE_DISABLE;			        /*指定是否启用TI模式 */
    SpiHandle.Init.Mode 			 = SPI_MODE_MASTER;		                /*设置SPI的主/从机模式,可选主机/从机 */
```

SPI it模式数据传输提供了3个API：

HAL_SPI_Transmit_IT：发送数据有效

HAL_SPI_Receive_IT： 接收数据有效

HAL_SPI_TransmitReceive_IT：发送和接收数据同时有效

```c
HAL_StatusTypeDef HAL_SPI_Transmit_IT(SPI_HandleTypeDef *hspi, uint8_t *pTxData, uint16_t Size)
HAL_StatusTypeDef HAL_SPI_Receive_IT(SPI_HandleTypeDef *hspi, uint8_t *pRxData, uint16_t Size)
HAL_StatusTypeDef HAL_SPI_TransmitReceive_IT(SPI_HandleTypeDef *hspi, uint8_t *pTxData, uint8_t *pRxData,
                                             uint16_t Size);
```

下面是HAL_SPI_TransmitReceive_IT API中各参数的说明：

```c
/**
  * @brief  Transmit and Receive an amount of data in non-blocking mode with Interrupt.
  * @param  hspi pointer to a SPI_HandleTypeDef structure that contains
  *               the configuration information for SPI module.
  * @param  pTxData pointer to transmission data buffer
  * @param  pRxData pointer to reception data buffer
  * @param  Size amount of data to be sent and received
  * @retval HAL status
  */
HAL_StatusTypeDef HAL_SPI_TransmitReceive_IT(SPI_HandleTypeDef *hspi, uint8_t *pTxData, uint8_t *pRxData, uint16_t Size)


```

数据传输完成时会调用callback函数：

```c
/* master device */
void HAL_SPI_CpltCallback(SPI_HandleTypeDef *hspi)
{
    SPI_CS_HIGH();
    /* Turn LED on: Transfer in transmission/reception process is correct */
    io_set_pin(LED_IO);
    ComState = COM_COMPLETE;
}

/* slave device */
void HAL_SPI_CpltCallback(SPI_HandleTypeDef *hspi)
{
    /* Turn LED on: Transfer in transmission/reception process is correct */
    io_set_pin(LED_IO);
    ComState = COM_COMPLETE;
}
```



#### 2.1.1 端口连接

例程需要使用两块开发板，在程序开始运行之前需要按照下面方式连接master和slave设备：

| SPI MASTER BOARD | SPI SLAVE BOARD |
| :--------------: | :-------------: |
|  spi_master_clk  |  spi_slave_clk  |
|  spi_master_cs   |  spi_slave_cs   |
| spi_master_mosi  | spi_slave_mosi  |
| spi_master_miso  | spi_slave_miso  |
|       GND        |       GND       |

#### 2.1.2  运行程序

主从机程序分别编译后下载到对应的开发板中，需要先在从机上进行复位，然后在主机上进行复位后观察开发板上LED的状态

### 2.2 测试结果

在本例程中，aTxBuffer是预定义的，aRxBuffer大小与aTxBuffer相同。SPI主机通过HAL_SPI_TransmitReceive_IT()发送aTxBuffer和接收RxBuffer的数据，同时SPI从机通过HAL_SPI_TransmitReceive_IT()发送TxBuffer和接收RxBuffer的数据，通过Buffercmp()比较aRxBuffer和aTxBuffer，以检查buffer中数据的正确性。 

板子上PA01对应的的LED可用于监控传输状态：

-当数据传输完成时LED会点亮

-当发送/接收过程中数据出现错误时，LED会以500ms的间隔闪烁