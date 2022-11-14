# ADC 单通道采样

例程路径：ls_sdk\examples\peripheral\adcv2\adc_single_channel

## 一、功能描述

本示例演示了使用中断方式分别对规则、注入、循环通道组的单通道采集。

## 二、软件配置

### 2.1 ADC IO 配置

例程中我们使用ADC0模块的通道5，使用以下函数配置相应IO的复用功能

```
pinmux_adc0_in5_init();// PA01
```

### 2.2 基本ADC采样配置

```
void ADC_Init_Func(void)
{
    ADCx_Hdl.Instance                   = LSADC;//选择硬件ADC模块
    ADCx_Hdl.Init.DataAlign             = ADC_DATAALIGN_RIGHT;//数据右对齐
    ADCx_Hdl.Init.ContinuousConvMode    = DISABLE;//单通道采样，不需要配连续模式
    ADCx_Hdl.Init.NbrOfConversion       = 1;//配置规则组转换序列长度
    ADCx_Hdl.Init.DiscontinuousConvMode = DISABLE;// 不使用间断转换模式
    ADCx_Hdl.Init.NbrOfDiscConversion   = 0; // 间断转换序列长度 
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

### 2.3 规则组通道配置

```
void ADC_RegMode_Channel_setCfg(void)
{
    ADC_ChannelConfTypeDef sConfig  = {0};
    sConfig.Channel         = ADC_CHANNEL_5;// ADC通道选择，与具体IO相对应
    sConfig.Rank            = ADC_REGULAR_RANK_1;// 采样序列配置，这里表示第一次采样的通道
    sConfig.SamplingTime    = ADC_SAMPLETIME_15CYCLES;// 采样周期
    sConfig.clk_cfg         = ADC_CH_CLOCK_DIV8;// 通道时钟配置
    if (HAL_ADC_ConfigChannel(&ADCx_Hdl, &sConfig) != HAL_OK)
    {
        Error_Handler();
    }
}

// 开始中断采集
HAL_StatusTypeDef HAL_ADC_Start_IT(ADC_HandleTypeDef *hadc);
// 采集完成后的中断回调，需要在回调函数里面去获取ADC的值
void HAL_ADC_ConvCpltCallback(ADC_HandleTypeDef* hadc);
```

### 2.4 注入组通道配置

```
void ADC_InjMode_Channel_setCfg(void)
{
    ADC_InjectionConfTypeDef sConfigInjected = {0};
    sConfigInjected.InjectedChannel         = ADC_CHANNEL_5;// ADC通道选择，与具体IO相对应
    sConfigInjected.InjectedRank            = ADC_INJECTED_RANK_1;// 采样序列配置，这里表示第一次采样的通道
    sConfigInjected.InjectedSamplingTime    = ADC_SAMPLETIME_15CYCLES;// 采样周期
    sConfigInjected.InjectedOffset          = 0; // 偏移值
    sConfigInjected.InjectedClk             = ADC_CH_CLOCK_DIV8;// 通道时钟配置
    sConfigInjected.InjectedNbrOfConversion = 1;//注入通道组采样序列长度
    sConfigInjected.InjectedDiscontinuousConvMode = DISABLE;//不使用间断模式
    if (HAL_ADCEx_InjectedConfigChannel(&ADCx_Hdl, &sConfigInjected) != HAL_OK)
    {
        Error_Handler();
    }
}

// 开始中断采集
HAL_StatusTypeDef HAL_ADCEx_InjectedStart_IT(ADC_HandleTypeDef* hadc);
// 采集完成后的中断回调，需要在回调函数里面去获取ADC的值
void HAL_ADC_ConvCpltCallback(ADC_HandleTypeDef* hadc);
```

### 2.5 循环组通道配置

```
void ADC_LoopMode_Channel_setcfg(void)
{
    ADC_LoopConfTypeDef sConfig = {0};
    sConfig.Channel         = ADC_CHANNEL_5;// ADC通道选择，与具体IO相对应
    sConfig.Rank            = ADC_LOOP_RANK_1;// 采样序列配置，这里表示第一次采样的通道
    sConfig.SamplingTime    = ADC_SAMPLETIME_15CYCLES;// 采样周期
    sConfig.LoopClk         = ADC_CH_CLOCK_DIV8;// 通道时钟配置
    sConfig.NbrOfConversion = 1;//循环通道组采样序列长度
    sConfig.CapIntv = 0xFFFF;//两次捕获序列之间的间隔，轮询模式此配置无效
    if(HAL_ADC_LoopConfigChannel(&ADCx_Hdl,&sConfig)!= HAL_OK)
    {
        Error_Handler();
    }
}

//开始中断采集 
HAL_StatusTypeDef HAL_ADCx_LoopStart_IT(ADC_HandleTypeDef *hadc);
// 采集完成后的中断回调，需要在回调函数里面去获取ADC的值
void HAL_ADCx_LoopConvCpltCallback(ADC_HandleTypeDef* hadc);
```

**对于循环组使用，应用上面只需要触发一次，在完成一轮采样后会自动继续下一轮的采样。为了避免出现FIFO溢出的情况，通常需要在回调函数内去获取FIFO中有几组数据，然后再将FIFO中数据全部读出。**

## 三、下载验证

将开发板的PA01输入电压，运行程序，打开RTT LOG查看打印信息。