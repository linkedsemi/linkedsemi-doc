# ADC 轮询采样

例程路径：ls_sdk\examples\peripheral\adcv2\adc_blocking_sampling

## 一、 功能描述

- ADC 轮询采样表示软件上面开始触发采样到电压采集完成是个顺序执行的动作,在应用上面这两个步骤之间不会插入其它操作，通常都是在一个函数内完成。
- 适用于规则通道、注入通道、循环通道。使用循环通道时，当一轮采集完时便会停止采样，直到下一次软件触发。

## 二、 软件配置

### 2.1 ADC IO配置

例程中我们使用ADC0模块的通道5、6、7三个通道，可以使用以下接口配置IO的复用功能。

```
static void pinmux_adc_init(void)
{
    pinmux_adc0_in5_init();// PA01
    pinmux_adc0_in6_init();// PA02
    pinmux_adc0_in7_init();// PA03
}
```

### 2.2 基本ADC采样配置

```
void ADC_Init_Func(void)
{
    ADCx_Hdl.Instance                   = LSADC; //选择硬件ADC模块
    ADCx_Hdl.Init.DataAlign             = ADC_DATAALIGN_RIGHT;//数据右对齐
    ADCx_Hdl.Init.ContinuousConvMode    = ENABLE;// 使能连续转换模式
    ADCx_Hdl.Init.NbrOfConversion       = 3;//配置规则组转换序列长度
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
    //省略不同通道的相同配置功能描述
}
```

- 规则组通道触发采样

```
void ADC_RegMode_Poll_GetValue(void)
{
	// 触发开始采样
    HAL_ADC_Start(&ADCx_Hdl);
    
    // 等待采样结束
    if (HAL_ADC_PollForConversion(&ADCx_Hdl, 1000) == HAL_OK)
    {
    	// 获取三个通道采样的结果
        LOG_I("RegMode:%d,%d,%d", HAL_ADC_GetValue(&ADCx_Hdl, ADC_REGULAR_RANK_1),
                          HAL_ADC_GetValue(&ADCx_Hdl, ADC_REGULAR_RANK_2),
                          HAL_ADC_GetValue(&ADCx_Hdl, ADC_REGULAR_RANK_3));
    }
    else
    {
        LOG_I("TIMEOUT");
    }
}
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
    sConfigInjected.InjectedNbrOfConversion = 3;//注入通道组采样序列长度
    sConfigInjected.InjectedDiscontinuousConvMode = DISABLE;//不使用间断模式
    if (HAL_ADCEx_InjectedConfigChannel(&ADCx_Hdl, &sConfigInjected) != HAL_OK)
    {
        Error_Handler();
    }
	//省略不同通道的相同配置功能描述
}

```

- 注入组通道触发采样

```
void ADC_InjMode_Poll_GetValue(void)
{
	// 触发开始采样
    HAL_ADCEx_InjectedStart(&ADCx_Hdl);
    // 等待采样结束
    if (HAL_ADCEx_InjectedPollForConversion(&ADCx_Hdl, 1000) == HAL_OK)
    {
    	// 获取三个通道采样的结果
        LOG_I("InjMode:%d,%d,%d", HAL_ADCEx_InjectedGetValue(&ADCx_Hdl, ADC_REGULAR_RANK_1),
                          HAL_ADCEx_InjectedGetValue(&ADCx_Hdl, ADC_REGULAR_RANK_2),
                          HAL_ADCEx_InjectedGetValue(&ADCx_Hdl, ADC_REGULAR_RANK_3));
    }
    else
    {
        LOG_I("TIMEOUT");
    }
}
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
    sConfig.NbrOfConversion = 3;//循环通道组采样序列长度
    sConfig.CapIntv = 0xFFFF;//两次捕获序列之间的间隔，轮询模式此配置无效
    if(HAL_ADC_LoopConfigChannel(&ADCx_Hdl,&sConfig)!= HAL_OK)
    {
        Error_Handler();
    }
	//省略不同通道的相同配置功能描述
}
```

- 循环组通道触发采样

```
void ADC_LoopMode_Poll_GetValue(void)
{
	// 触发开始采样
    HAL_ADCx_LoopStart(&ADCx_Hdl);
    // 等待采样结束
    if (HAL_ADCx_LoopPollForConversion(&ADCx_Hdl, 1000) == HAL_OK)
    {
    	// 获取三个通道采样的结果
        LOG_I("LoopMode:%d,%d,%d", HAL_ADCx_LoopGetValue(&ADCx_Hdl),
                        HAL_ADCx_LoopGetValue(&ADCx_Hdl),
                        HAL_ADCx_LoopGetValue(&ADCx_Hdl));
    }
    else
    {
        LOG_I("TIMEOUT");
    }
}
```

**循环组道通常是软件触发一次后，在完成最后一个通道采样后会自动进行下一轮的转换。当使用轮询模式时，获取完一组转换完成的数据后，驱动里面会主动将触发信号清零，直到下一次软件上面开始获取数据，才会开始新的一轮采样。**

## 三、下载验证

将开发板的PA01、PA02、PA03输入电压，运行程序，打开RTT LOG查看打印信息，三个通道的AD值，依次用规则通道、注入通道、轮询通道的方式捕获到并且打印出来。