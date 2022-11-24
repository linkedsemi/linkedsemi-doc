# 双DAC通道输出电压

例程路径：ls_sdk\examples\peripheral\dac\dual_dac_channel_voltage

## 一、功能描述

本示例使用软件触发方式实现了双DAC通道输出电压。

## 二、软件配置

### 2.1 DAC IO配置

DAC1对应的IO为PA07,DAC2对应的IO为PC04;

例程中使用两个DAC同时实现固定电压输出，以下函数配置相应IO的复用功能。

```c
    pinmux_dac1_init();         // PA07
    pinmux_dac2_init();         // PC04
```

### 2.2 基本DAC配置

```c
void DAC_Init_Func(void)
{
    DACx_Hdl.Instance           = LSDAC12;      // 选择硬件DAC模块，DAC1和DAC2的基址相同
    DACx_Hdl.DACx               = DAC1AndDAC2;  // 选择要使用的DAC

    //有关DAC1的配置
    DACx_Hdl.DAC1_Trigger       = SOFTWARE_TRIG;  // 选择触发器
    DACx_Hdl.DAC1_wave          = No_Wave;        // 无波形
    DACx_Hdl.DAC1_Mamp          = triangle_amplitude_4095;

    //有关DAC2的配置
    DACx_Hdl.DAC2_Trigger       = SOFTWARE_TRIG;  // 选择触发器
    DACx_Hdl.DAC2_wave          = No_Wave;        // 无波形
    DACx_Hdl.DAC2_Mamp          = triangle_amplitude_4095;

    if (HAL_DAC_Init(&DACx_Hdl) != HAL_OK)
    {
        Error_Handler();
    }
}
```

### 2.3 设置要转换成电压的数字量

```c
HAL_DAC_SetValue(&DACx_Hdl, DAC12_ALIGN_12B_RD, 0xfff0fff);
//注：高12位为DAC1的配置，低12位DAC2的配置，0没有含义。
```

## 三、 下载验证

编译运行程序，使用万用表测量分别测量PA07和PC04引脚处的电压，正常情况测到的电压均应该在1.87v左右。打开RTT LOG查看打印信息。
电压计算公式：DACoutput  =  (VOR /4095) * Vref