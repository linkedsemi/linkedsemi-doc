# 单DAC通道输出波形

例程路径：ls_sdk\examples\peripheral\dac\single_dac_channel_wave

## 一、功能描述

本示例使用定时器触发方式实现了单DAC通道的噪声波和三角波的产生。

## 二、软件配置

### 2.1 DAC IO配置

DAC1对应的IO为PA07,DAC2对应的IO为PC04;

例程中使用DAC2实现波形的产生，以下函数配置相应IO的复用功能。

```c
pinmux_dac2_init(); // PC04
```

### 2.2 定时器结构体配置

```c
void Basic_Timer_Cfg(void)
{ 
    TimHandle.Instance           = LSGPTIMA;        // 选择定时器
    TimHandle.Init.Prescaler     = TIM_PRESCALER;   // 预分屏器
    TimHandle.Init.Period        = TIM_PERIOD;      // 定时器计数值
    TimHandle.Init.ClockDivision = 0;               // 时钟分频，配置死去时间才会用到，忽略
    TimHandle.Init.CounterMode   = TIM_COUNTERMODE_DOWN;  // 计数模式
    TimHandle.Init.TrgoSource = TIM_TRGO_UPDATE;    //更新 – 更新事件被选为触发输入
    // 初始化定时器
    HAL_TIM_Init(&TimHandle);   
    HAL_TIM_Base_Start(&TimHandle);
}
```

### 2.3 基本DAC配置

```c
void DAC_Init_Func(void)
{
    DACx_Hdl.Instance           = LSDAC12;   // 选择硬件DAC模块，DAC1和DAC2的基址相同
    DACx_Hdl.DACx               = DAC2;      // 选择要使用的DAC
    DACx_Hdl.DAC2_Trigger       = GENERAL_TimerA_TRGO; // 选择触发器
    DACx_Hdl.DAC2_wave          = Triangle_Wave;        // 选择要产生的波形，此处设置为三角波
    DACx_Hdl.DAC2_Mamp          = triangle_amplitude_4095; // 选择波形的峰值
    if (HAL_DAC_Init(&DACx_Hdl) != HAL_OK)
    {
        Error_Handler();
    }
}
```

注：配置触发器和定时器的时候需要匹配。
例如：当 DACx_Hdl.DAC2_Trigger = GENERAL_TimerA_TRGO 时 TimHandle.Instance = LSGPTIMA;

## 三、下载验证

将示波器的探头接开发板的PC04引脚，编译运行程序，查看波形。