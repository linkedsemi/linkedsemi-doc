# 单DAC通道输出电压

例程路径：ls_sdk\examples\peripheral\dac\single_dac_channel_voltage

## 一、功能描述

本示例使用软件触发方式实现了单DAC通道输出电压。

## 二、软件配置

### 2.1 DAC IO配置

DAC1对应的IO为PA07,DAC2对应的IO为PC04;

例程中使用DAC2实现固定电压输出，以下函数配置相应IO的复用功能。

```c
pinmux_dac2_init(); // PC04
```

### 2.2 基本DAC配置

```c
void DAC_Init_Func(void)
{
    DACx_Hdl.Instance           = LSDAC12;   // 选择硬件DAC模块，DAC1和DAC2的基址相同
    DACx_Hdl.DACx               = DAC2;      // 选择要使用的DAC
    DACx_Hdl.DAC2_Trigger       = SOFTWARE_TRIG; // 选择触发器
    DACx_Hdl.DAC2_wave          = No_Wave;        // 选择要产生的波形，此处设置为无波形
    DACx_Hdl.DAC2_Mamp          = triangle_amplitude_1; // 选择波形的峰值
    if (HAL_DAC_Init(&DACx_Hdl) != HAL_OK)
    {
        Error_Handler();
    }
}
```

### 2.3 设置要转换成电压的数字量

```c
HAL_DAC_SetValue(&DACx_Hdl, DAC2_ALIGN_12B_R, 4095);

//   DACx_Hdl: DAC handle
//   DAC2_ALIGN_12B_R: 选择数据对齐方式，此处为12位右对齐。
//   4095：要转换成电压的数字量。
```

## 三、下载验证

编译运行程序，使用万用表测量开发板PC04引脚的电压，测到的电压为1.87v左右。打开RTT LOG查看打印信息。
电压计算公式：DACoutput  =  (VOR /4095) * Vref
