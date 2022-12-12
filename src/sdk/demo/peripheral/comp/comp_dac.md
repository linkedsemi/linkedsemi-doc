# COMP_DAC 使用示例

例程路径： ls_sdk/examples/peripheral/comp/comp_dac

## 一、程序基本配置及说明：
```c
static COMP_HandleTypeDef COMP_Config;
static void comp_init()
{
    pinmux_comp1_init(PA00);
    COMP_Config.COMP = LSCOMP1;
    HAL_COMP_Init(&COMP_Config);
}
```
- COMP1的输出IO初始化
- 配置COMP_Config中实例化对象为LSCOMP1
- 调用HAL_COMP_Init()进行COMP模块初始化

## 二、操作步骤
### 2.1 参数设置
本例程中设置的参考电压源为DAC2，时钟采用MSI，迟滞电压选择5.4mV，同时给上升沿及下降沿中断使能。
```c
static void test_comp1()
{
    COMP_Param param;
    param.risingintr_en         = ENABLE;
    param.fallingintr_en        = ENABLE;
    param.flt_byp               = COMP_FLT_ENABLE;
    param.flt_prd               = ENABLE;
    param.input                 = INPUT_COMP1_PC00;
    param.vrefsel               = VREFSEL_DAC2;
    param.vrefctl               = DISABLE;
    param.hysteresis            = HYS_MS_5P4MV;
    param.clk_mode              = MediumSpeed;
    HAL_COMP_Config(&COMP_Config, &param);
}
```
### 2.2 开始
```c
HAL_COMP_Start(&COMP_Config);
```
```c
void HAL_COMP_Callback(COMP_HandleTypeDef *hcomp, enum comp_intr_edge edge, bool output)
{
    uint8_t comp = 1;
    switch ((uint32_t)hcomp->COMP)
    {
    case (uint32_t)LSCOMP1:
        break;
    case (uint32_t)LSCOMP2:
        comp = 2;
        break;
    case (uint32_t)LSCOMP3:
        comp = 3;
        break;
    }
    switch (edge)
    {
    case COMP_EDGE_RISING:
        LOG_I("trigger : COMP%d--Rising   edge--Current level: %d", comp, output);
        break;
    case COMP_EDGE_FALLING:
        LOG_I("trigger : COMP%d--Falling  edge--Current level: %d", comp, output);
        break;
    case COMP_EDGE_BOTH:
        LOG_I("trigger : COMP%d--Both     edge--Current level: %d", comp, output);
        break;
    }
}
```
- HAL_COMP_Callback返回COMP实例、中断触发沿以及中断触发后的输出信号，进行判断然后输出Log
### 2.3 配置参考电压源 DAC
本例程选择的参考电压源为DAC2，初始化DAC2实例（必须是DAC2，不需要配置IO），配置DAC相关参数然后输出电压: 1V。
```c
#define DAC1_VOL 1.0
#define DAC1_CODE DAC1_VOL*4095/1.8

static void dac_init(void)
{
    DACx_Hdl.Instance           = LSDAC12;
    DACx_Hdl.DACx               = DAC2;
    DACx_Hdl.DAC2_Trigger       = SOFTWARE_TRIG;
    if (HAL_DAC_Init(&DACx_Hdl) != HAL_OK)
    {
        while (1);
    }
    HAL_DAC_SetValue(&DACx_Hdl,DAC2_ALIGN_12B_R,DAC1_CODE);
}
```
### 2.4 控制输入电压
COMP1的0号通道对应的pin为PC00，通过电源直接给PC00输入电压，根据情况对输入电压进行调节。

## 三 、测试结果
```c
I/NO_TAG:trigger : COMP1--Both     edge--Current level: 1
I/NO_TAG:trigger : COMP1--Both     edge--Current level: 1
I/NO_TAG:trigger : COMP1--Both     edge--Current level: 1
I/NO_TAG:trigger : COMP1--Both     edge--Current level: 1
I/NO_TAG:trigger : COMP1--Both     edge--Current level: 1
I/NO_TAG:trigger : COMP1--Both     edge--Current level: 1
I/NO_TAG:trigger : COMP1--Both     edge--Current level: 1
I/NO_TAG:trigger : COMP1--Both     edge--Current level: 0
I/NO_TAG:trigger : COMP1--Both     edge--Current level: 1
I/NO_TAG:trigger : COMP1--Both     edge--Current level: 0
I/NO_TAG:trigger : COMP1--Rising   edge--Current level: 1
I/NO_TAG:trigger : COMP1--Both     edge--Current level: 0
I/NO_TAG:trigger : COMP1--Rising   edge--Current level: 1
I/NO_TAG:trigger : COMP1--Falling  edge--Current level: 0
I/NO_TAG:trigger : COMP1--Rising   edge--Current level: 1
I/NO_TAG:trigger : COMP1--Both     edge--Current level: 0
I/NO_TAG:trigger : COMP1--Rising   edge--Current level: 1
```
- 开始时输入电压为0.9V，此时输出信号为 0
- 慢慢将输入电压提高到1.1V过程中，由于输入电压波动较大导致输出信号抖动不停触发中断
- 结束时输入电压为1.1V，此时输出信号为 1