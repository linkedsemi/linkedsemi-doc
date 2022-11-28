# 使用DMA实现单DAC通道输出正弦波

例程路径：ls_sdk\examples\peripheral\dac\single_dac_channel_sine_wave

## 一、 功能描述

DAC的DMA功能，实现单DAC通道输出正弦波

## 二、软件配置

### 2.1 DAC IO 配置

DAC1对应的IO为PA07,DAC2对应的IO为PC04;

例程中使用DAC2实现正弦波输出，以下函数配置相应IO的复用功能。

```c
pinmux_dac2_init(); // PC04
```

### 2.2 基本DAC配置

```c
void DAC_Init_Func(void)
{
    // DMA 初始化
    DMA_CONTROLLER_INIT(dmac1_inst);
    DACx_Hdl.Env.DMA.DMA_Channel = 1;
    DACx_Hdl.DMAC_Instance = &dmac1_inst;

    DACx_Hdl.Instance           = LSDAC12;  // 选择硬件DAC模块，DAC1和DAC2的基址相同
    DACx_Hdl.DACx               = DAC2;     // 选择要使用的DAC
    DACx_Hdl.DAC2_Trigger       = GENERAL_TimerA_TRGO;  // 选择触发器
    DACx_Hdl.DAC2_wave          = No_Wave;        // 选择要产生的波形，此处设置为无波形，构成波形的数据由用户自己定义。
    DACx_Hdl.DAC2_Mamp          = triangle_amplitude_4095;  // 选择波形的峰值
    if (HAL_DAC_Init(&DACx_Hdl) != HAL_OK)
    {
        Error_Handler();
    }
}
```

### 2.3 设置要输出的电压,启动DMA

在DAC的基础配置完成后，可以使用以下接口触发DAC转换，DAC要转换的数据由内存搬移到外设。

```c
HAL_DAC_Start_DMA(&DACx_Hdl,DAC2_ALIGN_12B_R,sin_wave,sizeof(sin_wave),HAL_DAC_ConvCpltCallback);

// sine_wave :构成正弦波的数组，可以在sine_wave.h文件中查看
```

## 三、下载验证

将示波器的探头接开发板的PC04引脚，编译运行程序，查看波形。