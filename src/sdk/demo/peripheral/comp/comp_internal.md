# COMP_INTERNAL 使用示例

例程路径： ls_sdk/examples/peripheral/comp/comp_internal

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
本例程中设置的参考电压源为内部基准电压0.9V，采用高速时钟，迟滞电压选择21.9mV，同时给上升沿及下降沿中断使能。
```c
static void test_comp1()
{
    COMP_Param param;
    param.risingintr_en         = ENABLE;
    param.fallingintr_en        = ENABLE;
    param.flt_byp               = COMP_FLT_ENABLE;
    param.flt_prd               = ENABLE;
    param.input                 = INPUT_COMP1_PC00;
    param.vrefsel               = VREFSEL_INTERNAL_REFERENCE_VOLTAGE;
    param.vrefctl               = VREFCTL_900MV;
    param.hysteresis            = HYS_HS_21P9MV;
    param.clk_mode              = HighSpeed;
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
### 2.3 控制输入电压
COMP1的0号通道对应的pin为PC00，通过电源直接给PC00输入电压，根据情况对输入电压进行调节。

## 三 、测试结果
```c
I/NO_TAG:trigger : COMP1--Rising   edge--1
I/NO_TAG:trigger : COMP1--Both     edge--0
I/NO_TAG:trigger : COMP1--Both     edge--0
I/NO_TAG:trigger : COMP1--Rising   edge--1
I/NO_TAG:trigger : COMP1--Falling  edge--0
I/NO_TAG:trigger : COMP1--Rising   edge--1
I/NO_TAG:trigger : COMP1--Both     edge--0
I/NO_TAG:trigger : COMP1--Rising   edge--1
I/NO_TAG:trigger : COMP1--Both     edge--0
I/NO_TAG:trigger : COMP1--Both     edge--0
I/NO_TAG:trigger : COMP1--Both     edge--0
I/NO_TAG:trigger : COMP1--Both     edge--0
I/NO_TAG:trigger : COMP1--Both     edge--0
I/NO_TAG:trigger : COMP1--Both     edge--0
I/NO_TAG:trigger : COMP1--Both     edge--0
I/NO_TAG:trigger : COMP1--Falling  edge--0
I/NO_TAG:trigger : COMP1--Both     edge--0
I/NO_TAG:trigger : COMP1--Both     edge--0
I/NO_TAG:trigger : COMP1--Both     edge--0
I/NO_TAG:trigger : COMP1--Both     edge--0
I/NO_TAG:trigger : COMP1--Both     edge--0
I/NO_TAG:trigger : COMP1--Rising   edge--1
I/NO_TAG:trigger : COMP1--Both     edge--0
I/NO_TAG:trigger : COMP1--Falling  edge--0
I/NO_TAG:trigger : COMP1--Rising   edge--1
I/NO_TAG:trigger : COMP1--Both     edge--0
I/NO_TAG:trigger : COMP1--Both     edge--0
I/NO_TAG:trigger : COMP1--Both     edge--0
I/NO_TAG:trigger : COMP1--Both     edge--0
I/NO_TAG:trigger : COMP1--Falling  edge--0
I/NO_TAG:trigger : COMP1--Rising   edge--1
```
- 开始时输入电压为0.6V，此时输出信号为 0
- 慢慢将输入电压提高到1.2V时，由于输入电压波动较大导致输出信号抖动不停触发中断
- 结束时输入电压为1.2V，此时输出信号为 1