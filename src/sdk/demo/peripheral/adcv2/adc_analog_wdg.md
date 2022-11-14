# ADC 模拟看门狗

例程路径：ls_sdk\examples\peripheral\adcv2\adc_analog_wdg

## 一、功能描述

- 如果ADC转换的模拟电压低于阈值下限或高于阈值上限，则模拟看门狗状态位会被置1，如果使能了中断则会产生中断。这些阈值可以在应用中进行配置。
- 只有规则通道和注入通道支持模拟看门狗功能。
- 可以选择监控单一通道或者多个通道的AD值。选则多通道监控时，只要其中一路的采样AD值不在设置的阈值范围内时，就会产生模拟看门狗事件中断。



## 二、软件配置

### 2.1 基本ADC采样配置

```
void ADC_Init_Func(void)
{
    ADCx_Hdl.Instance                   = LSADC; //选择硬件ADC模块
    ADCx_Hdl.Init.DataAlign             = ADC_DATAALIGN_RIGHT; //数据右对齐
    ADCx_Hdl.Init.ContinuousConvMode    = ENABLE; // 使能连续转换模式
    ADCx_Hdl.Init.NbrOfConversion       = 3; //配置规则组转换序列长度，与下面通道个数配置一致
    ADCx_Hdl.Init.DiscontinuousConvMode = DISABLE; // 不使用间断转换模式
    ADCx_Hdl.Init.NbrOfDiscConversion   = 0; // 间断转换序列长度 
    ADCx_Hdl.Init.TrigType              = ADC_SOFTWARE_TRIGT; // 触发方式，通常都是软件触发
    ADCx_Hdl.Init.Vref                  = ADC_VREF_INSIDE; // 参考源选择，这里选择内部参考
    ADCx_Hdl.Init.AdcDriveType          = BINBUF_DIRECT_DRIVE_ADC; // 采样路径选择
    ADCx_Hdl.Init.AdcCkDiv              = 256; // ADC 时钟分屏
    if (HAL_ADC_Init(&ADCx_Hdl) != HAL_OK)
    {
        Error_Handler();
    }
}
```

### 2.2 ADC 规则通道配置

```
void ADC_RegMode_Channel_setCfg(void)
{
    ADC_ChannelConfTypeDef sConfig  = {0};
    sConfig.Channel         = ADC_CHANNEL_5; // ADC通道选择，与具体IO相对应
    sConfig.Rank            = ADC_REGULAR_RANK_1; // 采样序列配置，这里表示第一次采样的通道
    sConfig.SamplingTime    = ADC_SAMPLETIME_15CYCLES; // 采样周期
    sConfig.clk_cfg         = ADC_CH_CLOCK_DIV8; // 通道时钟配置
    if (HAL_ADC_ConfigChannel(&ADCx_Hdl, &sConfig) != HAL_OK)
    {
        Error_Handler();
    }
	// 省略不同通道的相同配置功能描述
}
```

### 2.3 模拟看门狗配置

```
void HAL_ADC_Analog_Wdg_config(void)
{
    ADC_AnalogWDGConfTypeDef AnalogWDGConfig = {0};
    AnalogWDGConfig.WatchdogMode = ADC_ANALOGWATCHDOG_ALL_REG;
    AnalogWDGConfig.Channel = ADC_CHANNEL_0;
    AnalogWDGConfig.ITMode = ENABLE; // 使能模拟看门狗中断
    AnalogWDGConfig.HighThreshold = 2400; // 阈值上限
    AnalogWDGConfig.LowThreshold = 10; // 阈值下限
    HAL_ADC_AnalogWDGConfig(&ADCx_Hdl, &AnalogWDGConfig);
}
```

**说明：** WatchdogMode 和 Channel 参数配置是配合使用的，只有模拟看门狗模式选择单通道监控时配置的Channel才会生效。

### 2.4 事件触发

应用上面只需要触发ADC开始采样即可

```
HAL_StatusTypeDef HAL_ADC_Start_IT(ADC_HandleTypeDef *hadc);
```

### 2.5 事件回调

正常情况下当ADC转换完成后只会上转换完成的回调函数：

```
void HAL_ADC_ConvCpltCallback(ADC_HandleTypeDef *hadc);
```

如果模拟看门狗事件被触发的时候，应用上面会收到看门狗溢出回调函数:

```
void HAL_ADC_LevelOutOfWindowCallback(ADC_HandleTypeDef* hadc);
```

## 三、下载验证

将开发板的PA01、PA02、PA03输入电压，运行程序，打开RTT LOG查看打印信息，当调整其中一路的输入电压值让采到的AD值超过设置看门狗阈值上限，此时能看到看门狗溢出中断里面的打印。