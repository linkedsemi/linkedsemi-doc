# ADC 多通道DMA采样

例程路径：ls_sdk\examples\peripheral\adcv2\adc_multi_channel_dma

## 一、功能描述

- 对于ADC的DMA功能只有循环组通道才支持，规则通道和注入通道没有DMA功能。

## 二、软件配置

### 2.1 ADC IO配置

例程中我们使用ADC0模块的通道4、5、6、7四个通道，可以使用以下接口配置IO的复用功能。

```
static void pinmux_adc_init(void)
{
    pinmux_adc0_in4_init();// PA00
    pinmux_adc0_in5_init();// PA01
    pinmux_adc0_in6_init();// PA02
    pinmux_adc0_in7_init();// PA03
}
```

### 2.2 基本ADC采样配置

```
void ADC_Init_Func(void)
{
	//DMA初始化
    DMA_CONTROLLER_INIT(dmac1_inst);
    ADCx_Hdl.Env.DMA.DMA_Channel        = 0;
    ADCx_Hdl.DMAC_Instance              = &dmac1_inst;

    ADCx_Hdl.Instance                   = LSADC;//选择硬件ADC模块
    ADCx_Hdl.Init.DataAlign             = ADC_DATAALIGN_RIGHT;//数据右对齐
    ADCx_Hdl.Init.ContinuousConvMode    = DISABLE;//规则通道的配置项，此处忽略
    ADCx_Hdl.Init.NbrOfConversion       = 0;//规则通道的配置项，此处忽略
    ADCx_Hdl.Init.DiscontinuousConvMode = DISABLE;//规则通道的配置项，此处忽略
    ADCx_Hdl.Init.NbrOfDiscConversion   = 0;  // 间断转换序列长度,循环组忽略
    ADCx_Hdl.Init.TrigType              = ADC_SOFTWARE_TRIGT;// 触发方式，通常都是软件触发
    ADCx_Hdl.Init.Vref                  = ADC_VREF_INSIDE;// 参考源选择，这里选择内部参考
    ADCx_Hdl.Init.AdcDriveType          = BINBUF_DIRECT_DRIVE_ADC;// 采样路径选择
    ADCx_Hdl.Init.AdcCkDiv              = 256;// ADC 时钟分频，可以配置0~0x1FF
    if (HAL_ADC_Init(&ADCx_Hdl) != HAL_OK)
    {
        Error_Handler();
    }
}
```

### 2.3 循环组通道配置

```
void ADC_LoopMode_Channel_setcfg(void)
{
    ADC_LoopConfTypeDef sConfig = {0};
    sConfig.Channel         = ADC_CHANNEL_4;// ADC通道选择，与具体IO相对应
    sConfig.Rank            = ADC_LOOP_RANK_1;// 采样序列配置，这里表示第一次采样的通道
    sConfig.SamplingTime    = ADC_SAMPLETIME_15CYCLES;// 采样周期
    sConfig.LoopClk         = ADC_CH_CLOCK_DIV8;// 通道时钟配置
    sConfig.NbrOfConversion = 4;//循环通道组采样序列长度
    sConfig.CapIntv = 0xFFFF;//两次捕获序列之间的间隔，轮询模式此配置无效
    if(HAL_ADC_LoopConfigChannel(&ADCx_Hdl,&sConfig)!= HAL_OK)
    {
        Error_Handler();
    }
	
}
```

### 2.4 事件触发和中断回调

在ADC基础配置完成后，应用上面可以使用以下接口去触发ADC采样，ADC转换完成的结果会通过DMA搬移到指定内存中。

```
HAL_StatusTypeDef HAL_ADC_LoopChannel_Start_DMA(ADC_HandleTypeDef* hadc, uint16_t* pData, uint32_t Length,void (*Callback)(ADC_HandleTypeDef* hadc));
```

需要注意的是第三个参数长度的配置，这个长度可以是配置通道数的倍数关系。比如示例中使用了4个通道，如果Length配置为4的话，那完成4个通道的采集后，采样会停止；如果Length配置为8，那在采完4个通道的数据又会自动继续下一轮采样，直到采完8个数据才会停止。

当采完指定个数的AD值后，可以通过`void (*Callback)(ADC_HandleTypeDef* hadc)`回调函数获知。

## 三、下载验证

将开发板的PA00、PA01、PA02、PA03输入电压，运行程序，打开RTT LOG查看打印信息。