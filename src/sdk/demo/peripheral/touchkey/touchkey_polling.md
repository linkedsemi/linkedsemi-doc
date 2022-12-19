# TOUCHKEY_POLLING 使用示例

例程路径： ls_sdk/examples/peripheral/touchkey/touchkey_polling

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
    tkParam.scan_mtim = 0xfff; 
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
        sum[i] = sum[i] / count;
}
```
- sum是用于接收自校准结果数据
- count是单通道采集的数据次数，用以计算平均值作为参考
- 本函数可供参考，内部算法可以根据不同场景及需求修改

### 2.3	开始扫描

```c
 while (1)
    {
        HAL_TOUCHKEY_StartScan(data);
        for (uint8_t i = 0; i < 16; i++)
        {
            if ((tkParam.scan_channel_en & CO_BIT(i) && data[i] < chn_avg[i] / 10 * 6 && data[i] > chn_avg[i] / 10))
            {
                LOG_I("trigger CHANNEL:---%d---%d", i, data[i]);
            }
        }
        // DELAY_US(500*1000);//sleep 500ms
    }
```

- data 用于接收扫描结果数据
- chn_avg 存储着16个通道的参考值

根据通道使能情况对结果选择通道，然后根据chn_avg 进行判断是否存在接触,打印log如下：
```c
    I/NO_TAG:trigger CHANNEL:---4---3072
    I/NO_TAG:trigger CHANNEL:---4---3045
    I/NO_TAG:trigger CHANNEL:---4---3315
    I/NO_TAG:trigger CHANNEL:---5---3194
    I/NO_TAG:trigger CHANNEL:---5---3354
    I/NO_TAG:trigger CHANNEL:---5---3137
    I/NO_TAG:trigger CHANNEL:---6---3364
    I/NO_TAG:trigger CHANNEL:---6---3241
    I/NO_TAG:trigger CHANNEL:---6---2966
    I/NO_TAG:trigger CHANNEL:---6---3169
    I/NO_TAG:trigger CHANNEL:---6---3088
    I/NO_TAG:trigger CHANNEL:---7---3189
    I/NO_TAG:trigger CHANNEL:---7---3249
```


