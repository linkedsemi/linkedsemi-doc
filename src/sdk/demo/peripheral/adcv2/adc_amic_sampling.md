# AMIC 信号采集

例程路径：ls_sdk\examples\peripheral\adcv2\adc_amic_sampling

## 一、功能描述

本示例使用amic采集信号。

## 二、软件配置

### 2.1 Amic IO 配置

例程中我们使用Amic的专用通道，使用以下函数配置相应IO的复用功能。

```C
pinmux_amic_init(); //PB10
```

### 2.2 基本Amic采样配置

```C
void ADC_Init_Func(void)
{
    pinmux_amic_init();
    ADCx_Hdl.Instance                   = LSADC2;   //选择ADC2
    ADCx_Hdl.Init.DataAlign             = ADC_DATAALIGN_RIGHT;  //数据右对齐
    ADCx_Hdl.Init.ContinuousConvMode    = DISABLE;  //单通道采样，不需要置为连续模式
    ADCx_Hdl.Init.NbrOfConversion       = 1;  //配置循环模式转换序列长度
    ADCx_Hdl.Init.DiscontinuousConvMode = DISABLE;  //不使用间断转换模式
    ADCx_Hdl.Init.NbrOfDiscConversion   = 0;  // 间断转换序列长度
    ADCx_Hdl.Init.TrigType              = ADC_SOFTWARE_TRIGT;  //触发方式选择，通常为软件触发
    ADCx_Hdl.Init.Vref                  = ADC_VREF_INSIDE;     //参考源选择，这里选择内部参考电压
    ADCx_Hdl.Init.AdcDriveType          = INRES_ONETHIRD_EINBUF_DRIVE_ADC;   //采样路径选择，这里输入信号1/3分压之后，经过运放驱动ADC
    ADCx_Hdl.Init.AdcCkDiv              = 256;  //ADC 时钟分频，可以配置为0~0x1FF
    if (HAL_ADC_Init(&ADCx_Hdl) != HAL_OK)
    {
        Error_Handler();
    }
}
```

### 2.3 循环模式通道配置

```C
void ADC_LoopMode_Channel_setcfg(void)
{
    ADC_LoopConfTypeDef sConfig = {0};
    sConfig.Channel = ADC2_CHANNEL_AMIC;  //Amic通道选择，与具体io对应
    sConfig.Rank    = ADC_LOOP_RANK_1;   //采样序列配置，这里表示第一次采样的通道
    sConfig.SamplingTime = ADC_SAMPLETIME_15CYCLES;  //采样的周期
    sConfig.LoopClk      = ADC_CH_CLOCK_DIV1;  //通道时钟配置
    sConfig.NbrOfConversion = 1;  //循环模式采样序列长度
    sConfig.CapIntv = 0xffff;  //两次捕获序列之间的间隔，轮询模式此配置无效
    if(HAL_ADC_LoopConfigChannel(&ADCx_Hdl,&sConfig)!= HAL_OK)
    {
        Error_Handler();
    }
}

/*开始中断采集 */
HAL_StatusTypeDef HAL_ADCx_LoopStart_IT(ADC_HandleTypeDef *hadc);
/*采集完成后的中断回调，需要在回调函数里面去获取ADC的值*/
void HAL_ADCx_LoopConvCpltCallback(ADC_HandleTypeDef* hadc);
```

**对于循环组使用，应用上面只需要触发一次，在完成一轮采样后会自动继续下一轮的采样。为了避免出现FIFO溢出的情况，通常需要在回调函数内去获取FIFO中有几组数据，然后再将FIFO中数据全部读出。**

## 三、下载验证

设置ADC的信号输入通道为AMIC，对应io为PB10，控制DG4162信号发生器输出信号VPP = 2.5v,Voffet = 1.65V,信号输出频率为4.7KHZ，运行程序，打开RTT LOG查看打印信息。采样数据应是一个正弦波。