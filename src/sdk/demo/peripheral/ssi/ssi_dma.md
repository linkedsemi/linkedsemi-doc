# SSI_DMA 使用示例

例程路径: ls_sdk\examples\peripheral\ssi\ssi_dma

## 一、程序基本配置及说明：

ssi_dma示例通过dma的方式使用ssi传输数据，例程每10ms传输一组数据，在main中首先进行系统初始化和ssi初始化：

```c
        sys_init_none();
        ssi_test_init();
```

ssi IO端口设置：

```C
 	pinmux_ssi_clk_init(PB09);		/* CLK-------------PB09 */	
	pinmux_ssi_nss0_init(PB08);		/* SSN-------------PB08 */	
	pinmux_ssi_dq0_init(PA07);		/* MOSI------------PA07 */	
	pinmux_ssi_dq1_init(PA00);		/* MISO------------PA00 */
```



## 二、操作步骤及结果：

### 2.1 操作步骤

**说明**：

SSI初始化结构体配置说明：

```C
	SsiHandle.REG = LSSSI;										/*SSI 寄存器指针 */
	SsiHandle.Init.clk_div = 128;								/*设置时钟分频因子 */
	SsiHandle.Init.rxsample_dly = 0;							/*RX 采样延时 */
	SsiHandle.Init.ctrl.cph = SCLK_Toggle_In_Middle;			/*设置时钟相位CPH */
	SsiHandle.Init.ctrl.cpol = Inactive_Low;					/*设置时钟极性CPOL*/
	SsiHandle.Init.ctrl.data_frame_size = DFS_32_8_bits;		/*设置数据帧长度 */
```

SSI DMA模式数据传输提供了4个API：

HAL_SSI_Transmit_DMA：只能发送数据

HAL_SSI_Receive_DMA：  只能接收数据

HAL_SSI_TransmitReceive_DMA：可以同时发送和接收数据

HAL_SSI_TransmitReceive_HalfDuplex_DMA:  使用半双工的方式发送和接收数据

```c
HAL_SSI_Transmit_DMA(SSI_HandleTypeDef *hssi,void *Data,uint16_t Count)
HAL_SSI_Receive_DMA(SSI_HandleTypeDef *hssi,void *Data,uint16_t Count)
HAL_SSI_TransmitReceive_DMA(SSI_HandleTypeDef *hssi,void *TX_Data,void *RX_Data,uint16_t Count)
HAL_SSI_TransmitReceive_HalfDuplex_DMA(SSI_HandleTypeDef *hssi,void *TX_Data,uint16_t TX_Count,void *RX_Data,uint16_t RX_Count)
```

下面是HAL_SSI_TransmitReceive_DMA API中各参数的说明：

```c
/**
  * @brief  Transmit and Receive an amount of data in non-blocking mode with DMA
  * @param  hssi pointer to a SSI_HandleTypeDef structure that contains
  *               the configuration information for SPI module.
  * @param  TXData pointer to transmission data buffer
  * @param  RXData pointer to reception data buffer
  * @param  Count amount of data to be sent
  * @retval HAL status
  */
HAL_StatusTypeDef HAL_SSI_TransmitReceive_DMA(SSI_HandleTypeDef *hssi,void *TX_Data,void *RX_Data,uint16_t Count)


```

数据传输完成时会调用相应的callback函数：

```c
void HAL_SSI_TxRxDMACpltCallback(SSI_HandleTypeDef *hssi)
{
	LOG_I("RX CMPLT: 0x%x, 0x%x, 0x%x, 0x%x", ssi_dma_rx_buf[0],ssi_dma_rx_buf[1],ssi_dma_rx_buf[2],ssi_dma_rx_buf[3]);
	ssi_txrx_flag = 1;
	io_toggle_pin(PA01);
}
```

#### 2.1.1  运行程序

SSI例程采用本地回环测试，将MOSI和MISO端口短接，编译后下载到开发板中，然后对开发板进行复位后观察log打印信息

### 2.2 测试结果

在本例程中，ssi_tx_buf是预定义的，ssi_tx_buf大小与ssi_rx_buf相同。SSI通过HAL_SSI_TransmitReceive_DMA()发送ssi_tx_buf和接收ssi_rx_buf的数据，通过log打印信息检查接收数据的正确性。 

可通过log打印信息监控接收数据：

按照代码的配置，将MOSI和MISO短接（本地回环测试）

连接log，运行测试程序，在测试正常时，应该能持续看到TXRX CMPLT的打印数据,如果测试板上PA01有接LED灯，可以看到灯在持续闪烁



