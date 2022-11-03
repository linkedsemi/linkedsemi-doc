# TIMER 使用示例

## 一、定时

例程路径： ls_sdk/examples/peripheral/ timer/ Basic_TIM 

### 1.1 功能说明

Timer最基本的功能就是定时，该例程配置了一个250us周期的定时器，定时时间到了会产生一个定时器中断，然后在定时器中断里面执行IO翻转的动作。

### 1.2 软件配置

#### 1.2.1 定时时间计算

- 定时事件生成的时间主要由 TIMx_PSC 和 TIMx_ARR 两个寄存器值决定，这个也就是定时器的周期;
- 首先我们查看宏 SDK_HCLK_MHZ 配置的是多少，这决定我们系统时钟。timer的时钟来源于系统时钟（fCK_PSC），此时 fCK_PSC = SDK_HCLK_MHZ *1 MHz;
- fCK_PSC经过预分频器后输出时钟给到 CK_INT，也就是 TIMx_CNT 计数的时钟;
  - CK_INT = fCK_PSC /(PSC[15:0] + 1)
- 为了方便计算我们配置 CK_INT 时钟为 1MHz, 既 TIMx_CNT 每增加1个值的时间表示 1us，据此我们可以算出预分频器参数：TIMx_PSC = SDK_HCLK_MHZ  - 1；
- 确定  CK_CNT  为1us计数一次后，当需要250us定时周期时，则 TIMx_ARR = (250 - 1).

#### 1.2.2 定时器结构体配置

```
#define TIM_PRESCALER     (SDK_HCLK_MHZ-1)
#define TIM_PERIOD        (250 - 1)
TIM_HandleTypeDef TimHandle;

void Basic_Timer_Cfg(void)
{
	// 当TIMER上中断时，需要翻转IO的初始化
    io_cfg_output(PA00);
    io_write_pin(PA00,0);

    TimHandle.Instance           = LSGPTIMA;
    TimHandle.Init.Prescaler     = TIM_PRESCALER;      // 预分频器
    TimHandle.Init.Period        = TIM_PERIOD;         // 定时器计数值
    TimHandle.Init.ClockDivision = 0;                  // 时钟分频，配置死去时间才会用到，忽略
    TimHandle.Init.CounterMode   = TIM_COUNTERMODE_UP; // 计数模式
    
    // 初始化定时器
    HAL_TIM_Init(&TimHandle);
    // 开启定时器更新中断
    HAL_TIM_Base_Start_IT(&TimHandle);
}
```

#### 1.2.3 定时器中断处理

当定时器计数溢出产生更新中断时，驱动里面会从中断处理函数中上抛一个回调函数给到应用层，供用户处理。

```
void HAL_TIM_PeriodElapsedCallback(TIM_HandleTypeDef *htim)
{
    if(htim->Instance == LSGPTIMA)
    {
    	// 翻转IO
        io_toggle_pin(PA00);
    }
}
```

### 1.3 下载验证

保证开发板相关硬件连接正确，把编译好的程序下载到开发板。  使用示波器或者逻辑分析仪抓取PA00输出波形，可以看到PA00每隔250us翻转一下。

 

## 二、PWM 输出

例程路径： ls_sdk/examples/peripheral/ timer/Basic_PWM 

### 2.1 功能说明

PWM输出模式属于 TIMER 输出比较模式其中的一种，也是最常用一种PWM模式。例程中使用一个 TIMER 模块输出四种同一频率不同占空比的PWM波形。

PWM 输出就是对外输出脉宽（即占空比）可调的方波信号，信号频率由自动重装寄存器 ARR 的值决定，占空比由比较寄存器 CCR 的值决定。  

PWM 模式分为两种， PWM1 和 PWM2，总得来说是差不多，就看你怎么用而已 ，具体区别：

| 模式 | 计数器 CNT 计算方式 | 说明                                |
| ---- | ------------------- | ----------------------------------- |
| PWM1 | 递增                | CNT<CCR，通道 CH 为有效，否则为无效 |
|      | 递减                | CNT>CCR，通道 CH 为无效，否则为有效 |
| PWM2 | 递增                | CNT<CCR，通道 CH 为无效，否则为有效 |
|      | 递减                | CNT>CCR，通道 CH 为有效，否则为无效 |

但有效电平是高还是低，由  CCER 寄存器的 CCxP 位的值设定。

### 2.2 软件配置

#### 2.2.1 PWM 周期/占空比配置

- 与 TIMER 定时时间配置一样，PWM 由 TIMx_PSC 和 TIMx_ARR 两个寄存器值决定；

- 例程中 PWM 周期配成250us，既PWM频率为4KHz，因此可以算出：

  ```
  TIMx_PSC = SDK_HCLK_MHZ  - 1
  TIMx_ARR = 250 - 1
  ```

- 占空比的大小由TIMx_CCR 的值决定。假设需要设置占空比为50%，则 TIMx_CCR = TIMx_ARR * 50%;

#### 2.2.2 PWM IO配置

- 任意 IO 均可配置为 PWM 输出口，根据选用的TIMx以及对应的通道CHx将IO复用为PWM输出。例如，将PA00配置为LSGPTIMB的通道1输出:

  ```
  pinmux_gptimb1_ch1_init(PA00, true, 0);
  ```

#### 2.2.3 核心代码配置说明

```
#define TIM_PRESCALER     (SDK_HCLK_MHZ-1)
#define TIM_PERIOD        (250 - 1) 
#define TIM_PULSE1        125

TIM_HandleTypeDef TimHandle;
static void Basic_PWM_Output_Cfg(void)
{
    TIM_OC_InitTypeDef sConfig = {0};
    // 将IO复用为PWM输出口
    pinmux_gptimb1_ch1_init(PA00, true, 0);
   
    TimHandle.Instance = LSGPTIMB;
    TimHandle.Init.Prescaler = TIM_PRESCALER;  // 预分频器
    TimHandle.Init.Period = TIM_PERIOD;        // 定时器周期
    TimHandle.Init.ClockDivision = 0;          // 时钟分频
    TimHandle.Init.CounterMode = TIM_COUNTERMODE_UP;// 计数模式
    TimHandle.Init.AutoReloadPreload = TIM_AUTORELOAD_PRELOAD_DISABLE;
    HAL_TIM_Init(&TimHandle);
    
    sConfig.OCMode = TIM_OCMODE_PWM1;  // PWM1输出模式
    sConfig.OCPolarity = TIM_OCPOLARITY_HIGH; //有效电平
    sConfig.OCFastMode = TIM_OCFAST_DISABLE; // 比较输出模式快速使能
    sConfig.Pulse = TIM_PULSE1; // 比较输出脉冲宽度
    HAL_TIM_PWM_ConfigChannel(&TimHandle, &sConfig, TIM_CHANNEL_1);
    
    HAL_TIM_PWM_Start(&TimHandle, TIM_CHANNEL_1);
}

```

### 2.3 下载验证

保证开发板相关硬件连接正确，把编译好的程序下载到开发板。  使用示波器或者逻辑分析仪抓取PA00、PA01、PB14、PB15输出波形，可以看到有四个周期相同占空比不同的波形。

## 三、带死区嵌入的互补PWM输出

例程路径： ls_sdk/examples/peripheral/ timer/DTC_PWM

### 3.1 功能说明

这个例程是在PWM输出的基础上面新增带死去嵌入的互补PWM输出功能。通常在电机应用里面会用到，简单举例说明下，在半桥驱动电路里面，MOS管Q1导通，Q2截止，此时如果想让Q2导通Q1截止，肯定是要先让Q1截止一段时间之后，再等一段时间才让Q2导通，那么这个等待的时间就称为死区时间，因为受MOS管工艺决定关闭是需要时间的，如果Q1关闭之后，马上打开Q2，那么此时一段时间内相当于Q1和Q2都导通了，这样电路会短路。

### 3.2 软件配置

PWM相关配置说明前面已经说明过，不再赘述。这里主要讲下互补输出以及死区时间配置。

- 对于互补输出功能来说，只需要在原来PWM输出的基础上面增加两个配置

```
pinmux_adtim1_ch1n_init(PA01); //互补输出引脚配置
HAL_TIMEx_PWMN_Start(&TimHandle, TIM_CHANNEL_1);//互补通道输出
```

- 死去嵌入的功能只需要在开始PWM输出之前将TIM_BreakDeadTimeConfigTypeDef结构体初始化好，然后再调用HAL_TIMEx_ConfigBreakDeadTime函数将相关配置写入寄存中即可，死区时间的计算可以参考TIMx_BDTR寄存器中的DTG[7:0]相关描述。

### 3.3 下载验证

保证开发板相关硬件连接正确，把编译好的程序下载到开发板。通过示波器或者逻辑分析仪抓取PA00和PA01上输出波形，可以看到两个互补的波形输出，并且两个波形的上升沿和下降沿之间会有一段Delay时间。

## 四、输入捕获

例程路径： ls_sdk/examples/peripheral/ timer/Basic_PWM 

输入捕获一般有两种常见应用，一种是用来检测脉冲边沿跳变的时间，另外一种是对PWM输入的测量可以很快知道PWM的占空比以及周期值。

### 4.1 脉宽测量

#### 4.1.1 功能描述

- 软件上面设置需要捕获的第一个边沿，例如配置为上升沿，当发生第一次捕获的时候，计数器CNT值会被锁存到捕获寄存器CCR中，  而且还会进入捕获中断，在中断服务程序中记录一次捕获，并且保存CCR的值，并且切换捕获的边沿为下降沿，这样当下次下降来的时候就会发生第二次捕获中断，同样捕获的计数CNT值会被锁存到CCR中，这样两次CCR的差值就是当前脉冲宽度的值了。

  ```
  // 当发生捕获中断的时候应用的回调函数
  void HAL_TIM_IC_CaptureCallback(TIM_HandleTypeDef *htim)
  ```

- 在测量脉宽过程中需要来回的切换捕获边沿的极性，如果测量的脉宽时间比较长，定时器就会发生溢出，溢出的时候会产生更新中断，我们可以在中断里面对溢出进行记录处理。 

  ```
  // 发生定时器计数溢中断时应用上的回调函数
  void HAL_TIM_PeriodElapsedCallback(TIM_HandleTypeDef *htim);
  ```

#### 4.1.2 软件配置

- 配置需要捕获的脉冲信号，PA00 输出5ms周期占空比50%的PWM信号。

  ```
  static void PWM_Output_Config(void)
  {
      pinmux_gptima1_ch1_init(PA00,true,0);
      TIM_OC_InitTypeDef  TIM_OCInitStructure = {0};  
  
      TIM_PWMOUTPUT_Handle.Instance = LSGPTIMA;
      TIM_PWMOUTPUT_Handle.Init.Period = 5000-1;
      TIM_PWMOUTPUT_Handle.Init.Prescaler = TIM_PRESCALER;	
      TIM_PWMOUTPUT_Handle.Init.CounterMode = TIM_COUNTERMODE_UP;
      TIM_PWMOUTPUT_Handle.Init.ClockDivision=TIM_CLOCKDIVISION_DIV1;
      HAL_TIM_Init(&TIM_PWMOUTPUT_Handle);
  
      TIM_OCInitStructure.OCMode = TIM_OCMODE_PWM1;
      TIM_OCInitStructure.Pulse = 2500;
      TIM_OCInitStructure.OCFastMode = TIM_OCFAST_DISABLE;
      TIM_OCInitStructure.OCPolarity = TIM_OCPOLARITY_HIGH;	
  
      HAL_TIM_PWM_ConfigChannel(&TIM_PWMOUTPUT_Handle, &TIM_OCInitStructure, TIM_CHANNEL_1);
      HAL_TIM_PWM_Start(&TIM_PWMOUTPUT_Handle,TIM_CHANNEL_1);
  }
  ```

- 捕获timer的配置，主要是对TIM_IC_InitTypeDef 结构体成员变量进行配置

  ```
  pinmux_gptimc1_ch1_init(PA07, false, 0);//配置捕获信号输入的IO
  
  TIM_ICInitStructure.ICPolarity = TIM_ICPOLARITY_RISING; // 输入捕获触发选择
  TIM_ICInitStructure.ICSelection = TIM_ICSELECTION_DIRECTTI;// 输入捕获选择
  TIM_ICInitStructure.ICPrescaler = TIM_ICPSC_DIV1;// 输入捕获预分频器
  TIM_ICInitStructure.ICFilter = 0; // 输入捕获滤波器
  
  // 将配置写入到寄存器中
  HAL_TIM_IC_ConfigChannel(&TIM_Capture_Handle, &TIM_ICInitStructure, TIM_CHANNEL_1);
  
  // 启用捕获timer中断，用于记录捕获过程中8产生的溢出事件
  HAL_TIM_Base_Start_IT(&TIM_Capture_Handle); 
  // 启用捕获中断，当发生捕获事件时产生中断
  HAL_TIM_IC_Start_IT(&TIM_Capture_Handle,TIM_CHANNEL_1);
  ```

- 当发生捕获中断时，我们需要在中断里面去获取捕获的值，并且配置好下一次捕获的信号，可以使用HAL_TIM_ReadCapturedValue 接口获取捕获的计数值。

#### 4.1.3 下载验证

保证开发板相关硬件连接正确，把编译好的程序下载到开发板，用杜邦线把PA00和PA07相连，通过log查看输出的捕获的脉冲宽度值。

### 4.2 PWM输入模式

#### 4.2.1 功能描述

我们用一个定时器来产生已知频率和占空比的 PWM 信号，然后用另外一个定时器的 PWM 输入模式来测量这个已知的 PWM 信号的频率和占空比，通过两者的对比即可知道测量是否准确。  

#### 4.2.2 软件配置

- 部分配置代码说明：

```
	TIM_IC_InitTypeDef TIM_ICInitStructure = {0};
    pinmux_gptimc1_ch1_init(PA07, false, 0);
    // 配置IC1捕获：上升沿触发 TI1FP1
	TIM_ICInitStructure.ICPolarity = TIM_ICPOLARITY_RISING;
    TIM_ICInitStructure.ICSelection = TIM_ICSELECTION_DIRECTTI;
    TIM_ICInitStructure.ICPrescaler = TIM_ICPSC_DIV1;
    TIM_ICInitStructure.ICFilter = 0;
    HAL_TIM_IC_ConfigChannel(&TIM_Capture_Handle, &TIM_ICInitStructure, TIM_CHANNEL_1);
	// 配置IC2捕获：上升沿触发 TI1FP2
	TIM_ICInitStructure.ICPolarity = TIM_ICPOLARITY_FALLING;
    TIM_ICInitStructure.ICSelection = TIM_ICSELECTION_INDIRECTTI;
    TIM_ICInitStructure.ICPrescaler = TIM_ICPSC_DIV1;
    TIM_ICInitStructure.ICFilter = 0;
    HAL_TIM_IC_ConfigChannel(&TIM_Capture_Handle, &TIM_ICInitStructure, TIM_CHANNEL_2);
	// 选择从模式：复位模式
    TIM_SlaveConfigTypeDef TIM_SlaveConfigStructure = {0};
    TIM_SlaveConfigStructure.SlaveMode = TIM_SLAVEMODE_RESET;
    TIM_SlaveConfigStructure.InputTrigger = TIM_TS_TI1FP1;//选择触发输入
    HAL_TIM_SlaveConfigSynchro(&TIM_Capture_Handle, &TIM_SlaveConfigStructure);

    HAL_TIM_IC_Start_IT(&TIM_Capture_Handle, TIM_CHANNEL_1);
    HAL_TIM_IC_Start_IT(&TIM_Capture_Handle, TIM_CHANNEL_2);
```

- 捕获中断处理

  ```
  	if (htim->Channel == HAL_TIM_ACTIVE_CHANNEL_1)
      {
      	//获取输入捕获的值
          IC1Value = HAL_TIM_ReadCapturedValue(&TIM_Capture_Handle, TIM_CHANNEL_1);
          IC2Value = HAL_TIM_ReadCapturedValue(&TIM_Capture_Handle, TIM_CHANNEL_2);
          if (IC1Value != 0)
          {
          	// 占空比计算
              DutyCycle = ((IC2Value + 1) * 100) / (IC1Value + 1);
              // 频率计算
              Frequency = 1000000 / (IC1Value + 1);
          }
          else
          {
              DutyCycle = 0;
              Frequency = 0;
          }
      }
  ```

#### 4.2.3 下载验证

保证开发板相关硬件连接正确，把编译好的程序下载到开发板，用杜邦线把PA00和PA07相连，通过log查看输出的捕获的PWM频率和占空比的值。







