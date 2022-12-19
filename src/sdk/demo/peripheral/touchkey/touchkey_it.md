# TOUCHKEY_IT 使用示例

例程路径： ls_sdk/examples/peripheral/touchkey/touchkey_it

## 一、程序基本配置及说明：

```c
    /*TOUCHKEY模块初始化*/
    HAL_TOUCHKEY_Init();
    /*-----CMOD Init---*/
    pinmux_touchkey_cmod_init();
    /*--OPEN CH4,CH5,CH6,CH7--*/
    pinmux_touchkey_ch4_init();
    pinmux_touchkey_ch5_init();
    pinmux_touchkey_ch6_init();
    pinmux_touchkey_ch7_init();
```

## 二、操作步骤及结果：

### 2.1	扫描参数配置

```c
static TOUCHKEY_PARAM tkParam;

static void set_config()
{
  /*-------touchKey config params---------*/
    tkParam.clk_hprd = 0xf;
    tkParam.clk_lprd = 0xf;
    tkParam.scam_disch_prd = 0x7;
    tkParam.scan_channel_en = 0xf0;
    tkParam.scan_flt_prd = 0x3; 
    tkParam.scan_iter = 0xf;
    tkParam.scan_mtim = 0xfffff; 
    tkParam.touchkey_cp_vctl = TOUCHKEY_VCTRL_1800mV;
    /*-----Contains one SelfCalibration-----*/
    HAL_TOUCHKEY_SetParam(&tkParam);
}
```
[Periphera/touchkey](../../../peripheral/touchkey.rst)

### 2.2 自校准
```c
static void SelfCalibration(uint8_t count, uint32_t sum[16])
{
    uint8_t index = count;
    do
    {
        uint32_t result[16];
        HAL_TOUCHKEY_StartScan(result);
        for (uint8_t i = 0; i < 16; i++)
        {
            if (tkParam.scan_channel_en & CO_BIT(i))
            {
                sum[i] += result[i];
            }
        }
    } while (index--);
    for (uint8_t i = 0; i < 16; i++)
    {
        sum[i] = sum[i] / count;
    }
}
```
- sum是用于接收自校准结果数据
- count是单通道采集的数据次数，用以计算平均值作为参考
- 本函数可供参考，内部算法可以根据不同场景及需求修改

### 2.3	开始扫描

```c
int main()
{
 while (1)
    {
        flag = true;
        HAL_TOUCHKEY_StartScan_IT();
        while (flag)
            ;
        // DELAY_US(500*1000);
    }
}

void HAL_TOUCHKEY_END_Callback(uint32_t *data)
{
    for (uint8_t i = 0; i < 16; i++)
    {
        if ((tkParam.scan_channel_en & CO_BIT(i) && data[i] < chn_avg[i] / 10 * 6 && data[i] > chn_avg[i] / 10))
        {
            LOG_I("trigger CHANNEL:---%d---%d", i, data[i]);
        }
    }
    flag = false;
}
```

- data用于接收扫描结果数据
- falg用于判断扫描是否完成
- chn_avg 存储着16个通道的参考值

在回调函数中根据通道使能情况对结果选择通道，然后根据tkParam.normol进行判断是否存在接触,打印log如下：
```c
    I/NO_TAG:trigger CHANNEL:---4---3072
    I/NO_TAG:trigger CHANNEL:---4---3174
    I/NO_TAG:trigger CHANNEL:---4---3251
    I/NO_TAG:trigger CHANNEL:---5---3307
    I/NO_TAG:trigger CHANNEL:---5---3090
    I/NO_TAG:trigger CHANNEL:---5---3217
    I/NO_TAG:trigger CHANNEL:---6---3121
    I/NO_TAG:trigger CHANNEL:---6---3164
    I/NO_TAG:trigger CHANNEL:---6---2986
    I/NO_TAG:trigger CHANNEL:---6---3250
    I/NO_TAG:trigger CHANNEL:---6---3164
    I/NO_TAG:trigger CHANNEL:---7---3242
    I/NO_TAG:trigger CHANNEL:---7---3326
```


