# 使用DMA实现单DAC通道输出电压

例程路径：ls_sdk\examples\peripheral\dac\single_dac_channel_dma_voltage

## 一、功能描述

DAC的DMA功能，实现单DAC通道电压输出

## 二、软件配置

### 2.1 DAC IO 配置

DAC1对应的IO为PA07,DAC2对应的IO为PC04;

例程中使用DAC2实现固定电压输出，以下函数配置相应IO的复用功能。

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
    DACx_Hdl.DAC2_Trigger       = SOFTWARE_TRIG;  // 选择触发器
    DACx_Hdl.DAC2_wave          = No_Wave;        // 选择要产生的波形，此处设置为无波形
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
HAL_DAC_Start_DMA(&DACx_Hdl,DAC2_ALIGN_12B_R,single_dac_channel_value,sizeof(single_dac_channel_value),HAL_DAC_ConvCpltCallback);
```

说明：DMA_RAM_ATTR uint32_t single_dac_channel_value[DAC_CHANNLE_NUM]={0xfff,0x800,0x400};此数组用来存放要转换成电压的数字量。

## 三、下载验证

编译运行程序，使用万用表测量开发板PCO4引脚的电压，测到的电压大约分别为1.87v、0.96v、0.5v，使用示波器查看会更加直观。打开RTT LOG查看打印信息。
电压计算公式：DACoutput  =  (VOR /4095) * Vref