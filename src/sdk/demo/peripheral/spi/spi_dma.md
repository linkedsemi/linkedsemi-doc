# SPI_DMA 使用示例

例程路径: ls_sdk\examples\peripheral\spi\spi_dma

## 一、程序基本配置及说明：

spi_dma示例程序演示了使用DMA的方式实现spi master和spi slave之间的数据传输，例程以100ms的间隔传输10组数据

程序开始时先进行系统初始化和spi初始化：

```c
	/* system init app     */
        sys_init_none();
  	/* init spi and GPIO   */
        spi_init();
```

spi IO端口设置：

```C
      /* Configure the GPIO AF */
      /* CLK-------------PB12 */	
      /* SSN-------------PB13 */	
      /* MOSI------------PB14 */	
      /* MISO------------PB15 */	
  #ifdef 	MASTER_BOARD
    pinmux_spi2_master_clk_init(PB12);
    pinmux_spi2_master_nss_init(PB13);
    pinmux_spi2_master_mosi_init(PB14);
    pinmux_spi2_master_miso_init(PB15);
  #else
    pinmux_spi2_slave_clk_init(PB12);
    pinmux_spi2_slave_nss_init(PB13);
    pinmux_spi2_slave_mosi_init(PB14);
    pinmux_spi2_slave_miso_init(PB15);
  #endif
```

## 二、操作步骤及结果：

### 2.1 操作步骤

**说明**：

SPI初始化结构体配置说明：

```C
/* Set the SPI parameters */
  SpiHandle.Instance               = SPI2;   						          /*选择SPI Instance */
  SpiHandle.Init.BaudRatePrescaler = SPI_BAUDRATEPRESCALER_64;		/*设置时钟分频因子，fpclk/分频数=fSCK */
  SpiHandle.Init.Direction         = SPI_DIRECTION_2LINES;        /*设置SPI的单双向模式 */
  SpiHandle.Init.CLKPhase          = SPI_PHASE_1EDGE;				      /*设置时钟相位，可选奇/偶数边沿采样 */
  SpiHandle.Init.CLKPolarity       = SPI_POLARITY_LOW;				    /*设置时钟极性CPOL，可选高/低电平*/
  SpiHandle.Init.DataSize          = SPI_DATASIZE_8BIT;				    /*设置SPI的数据帧长度，可选8/16位 */
  SpiHandle.Init.FirstBit          = SPI_FIRSTBIT_MSB;				    /*设置MSB/LSB先行 */
  SpiHandle.Init.TIMode            = SPI_TIMODE_DISABLE;			    /*指定是否启用TI模式 */
  SpiHandle.Init.NSS               = SPI_NSS_HARD_OUTPUT;			    /*设置NSS引脚由SPI硬件控制还是软件控制*/
#ifdef MASTER_BOARD
  SpiHandle.Init.Mode 						= SPI_MODE_MASTER;		          /*设置SPI的主/从机模式 */
#else
  SpiHandle.Init.Mode						= SPI_MODE_SLAVE;
#endif /* MASTER_BOARD */
```

SPI DMA模式数据传输提供了3个API：

HAL_SPI_Transmit_DMA：只能发送数据

HAL_SPI_Receive_DMA：  只能接收数据

HAL_SPI_TransmitReceive_DMA：可以同时发送和接收数据 （全双工）

```c
HAL_SPI_Transmit_DMA(&SpiHandle, (uint8_t *)aTxBuffer, BUFFERSIZE)
HAL_SPI_Receive_DMA(&SpiHandle, (uint8_t *)aRxBuffer, BUFFERSIZE)
HAL_SPI_TransmitReceive_DMA(&SpiHandle,(uint8_t *)aTxBuffer, (uint8_t *)aRxBuffer,BUFFERSIZE)
```

下面是HAL_SPI_TransmitReceive_DMA  API中各参数的说明：

```c
/**
  * @brief  Transmit and Receive an amount of data in non-blocking mode with DMA.
  * @param  hspi pointer to a SPI_HandleTypeDef structure that contains
  *               the configuration information for SPI module.
  * @param  TXData pointer to transmission data buffer
  * @param  RXData pointer to reception data buffer
  * @param  Count amount of data to be sent
  * @retval HAL status
  */
HAL_StatusTypeDef HAL_SPI_TransmitReceive_DMA(SPI_HandleTypeDef *hspi,void *TX_Data,void *RX_Data,uint16_t Count)


```

数据传输完成时会调用相应的callback函数：

```c
void HAL_SPI_TxDMACpltCallback(SPI_HandleTypeDef *hspi)
{
  /* Turn LED on: Transfer in transmission/reception process is correct */
  io_set_pin(LED_IO);
  ComState = COM_COMPLETE;
}

void HAL_SPI_TxRxDMACpltCallback(SPI_HandleTypeDef *hspi) 
{
  /* Turn LED on: Transfer in transmission/reception process is correct */
  io_set_pin(LED_IO);
  ComState = COM_COMPLETE;
}

void HAL_SPI_RxDMACpltCallback(SPI_HandleTypeDef *hspi) 
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
|  spi_master_nss  |  spi_slave_nss  |
| spi_master_mosi  | spi_slave_mosi  |
| spi_master_miso  | spi_slave_miso  |
|       GND        |       GND       |

#### 2.1.2  运行程序

例程使用同一套代码实现spi主从机程序，通过宏定义"#define MASTER_BOARD" 选择主从机：

```c
/* Uncomment this line to use the board as master, if not it is used as slave */
#define MASTER_BOARD
```

 如果是主机则保留上面宏定义"#define MASTER_BOARD"，从机则需要注释掉上面的宏定义"#define MASTER_BOARD"

主从机程序分别编译后下载到对应的开发板中，需要先在从机上进行复位，然后在主机上进行复位后观察开发板上LED的状态

### 2.2 测试结果

在本例程中，aTxBuffer是预定义的，aRxBuffer大小与aTxBuffer相同。SPI主机通过HAL_SPI_TransmitReceive_DMA()发送aTxBuffer和接收RxBuffer的数据，同时SPI从机通过HAL_SPI_TransmitReceive_DMA()发送TxBuffer和接收RxBuffer的数据，通过Buffercmp()比较aRxBuffer和aTxBuffer，以检查buffer中数据的正确性。 

板子上PA01对应的的LED可用于监控传输状态：

-当数据传输完成时LED会点亮

-当发送/接收过程中数据出现错误时，LED会以500ms的间隔闪烁