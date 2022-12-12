# CAN 使用示例

## 一、回环测试

例程路径：ls_sdk\examples\peripheral\can\CAN_LoopBack

### 1.1、功能说明

本示例演示了如何使用单个开发板来验证CAN收发功能。回环模式下，芯片自发自收，可以通过逻辑分析仪抓取到 CAN_TX 引脚输出的波形，CAN_RX 不会接收外部输入的信号。

硬件上面 CAN_TX\CAN_RX 悬空即可，不需要接其它任何器件。

### 1.2、软件配置

- 初始化IO

  ```c
  void pinmux_bxcan_init(uint8_t txd,uint8_t rxd);
  ```

- 配置CAN外设的工作模式、位时序以及波特率

  ```c
  static void CAN_Mode_config(void)
  {
      Can_Handle.Instance = BXCAN;
      Can_Handle.pTxMsg = &TxMessage;
      Can_Handle.pRxMsg = &RxMessage;
  
      //关闭时间触发通信模式使能
      Can_Handle.Init.TTCM = DISABLE;
      //自动离线管理
      Can_Handle.Init.ABOM = ENABLE;
      //使用自动唤醒模式
      Can_Handle.Init.AWUM = ENABLE;
      //禁止报文自动重传 DISABLE-打开自动重传
      Can_Handle.Init.NART = DISABLE;
      //接收 FIFO 锁定模式 DISABLE-溢出时新报文会覆盖原有报文
      Can_Handle.Init.RFLM = DISABLE;
      //发送 FIFO 优先级 DISABLE-优先级取决于报文标示符
      Can_Handle.Init.TXFP = DISABLE;
      //使用回环模式
      Can_Handle.Init.Mode = CAN_MODE_LOOPBACK;
      
      //重新同步跳跃宽度1个时间单元
      Can_Handle.Init.SJW = CAN_SJW_1TQ;
  	/* ss=1 bs1=4 bs3=3 位时间宽度为 (1+4+3)=8tq */
      //时间段1占用了4个时间单元
      Can_Handle.Init.BS1 = CAN_BS1_4TQ;
      //时间段2占用了2个时间单元
      Can_Handle.Init.BS2 = CAN_BS2_3TQ;
      //分频系数配成7，实际为（7+1）=8分频
      Can_Handle.Init.Prescaler = 7;
      /* CAN的时钟频率为 SDK_HCLK_MHZ/2=32MHz,
         CAN波特率：32/(1+4+3)/(7+1) = 500 Kbps*/
      
      if(HAL_CAN_Init(&Can_Handle) != HAL_OK)
      {
          LOG_I("CAN INIT ERROR");
          while(1);
      }
  }
  ```

- 配置筛选器的工作方式

```c
static void CAN_Filter_Config(void)
{
    CAN_FilterConfTypeDef CAN_FilterInitStructure = {0};
    uint32_t flt_id = 0;

    //使用筛选器组0
    CAN_FilterInitStructure.FilterNumber = 0;
    //工作在掩码模式
    CAN_FilterInitStructure.FilterMode = CAN_FILTERMODE_IDLIST; 
    //筛选器位宽为单个32位
    CAN_FilterInitStructure.FilterScale = CAN_FILTERSCALE_32BIT; 

    //配置要筛选的ID
    flt_id = CAN_Fill_32Bit_FilterId(CAN_ID,CAN_IDE,CAN_RTR_DATA);
    //要筛选的ID高16位
    CAN_FilterInitStructure.FilterIdHigh = (flt_id & 0xFFFF0000) >> 16;
    //要筛选的ID低16位
    CAN_FilterInitStructure.FilterIdLow = flt_id & 0xFFFF;
    //筛选器高16位每位必须匹配
    CAN_FilterInitStructure.FilterMaskIdHigh = 0xFFFF; 
    //筛选器低16位每位必须匹配
    CAN_FilterInitStructure.FilterMaskIdLow = 0xFFFF;
    //筛选器被关联到 FIFO0
    CAN_FilterInitStructure.FilterFIFOAssignment = CAN_FILTER_FIFO0;
    //使能筛选器
    CAN_FilterInitStructure.FilterActivation = ENABLE;
    HAL_CAN_ConfigFilter(&Can_Handle, &CAN_FilterInitStructure);
}
```

- 设置发送的报文

  按照接收筛选的要求，填写好需要发送的报文后，调用  HAL_CAN_Transmit_IT  / HAL_CAN_Transmit 即可把该报文存储到发送邮箱，然后通过 CAN 外设发送出去。

- 接收报文

  - 中断方式接收

    提前通过调用 HAL_CAN_Receive_IT 来配置准备接收，当CAN接收的报文经过筛选器后会被存储到指定的FIFO中，并产生中断，应用上面会收到回调函数

    ```c
    void HAL_CAN_RxCpltCallback(CAN_HandleTypeDef* hcan)
    {
        uint32_t id_tmp = 0, ide_temp = 0, dlc_tmp = 0;
        ide_temp = hcan->pRxMsg->IDE;
        dlc_tmp = hcan->pRxMsg->DLC;
        if (ide_temp == CAN_ID_EXT)
        {
            id_tmp = hcan->pRxMsg->ExtId;
        }
        else
        {
            id_tmp = hcan->pRxMsg->StdId;
        }
        LOG_I("ID:0x%x,IDE:%d,DLC:%d", id_tmp, ide_temp, dlc_tmp);
        LOG_I("\r\nCAN IT RECV DATA:");
        LOG_HEX(Can_Handle.pRxMsg->Data,8);
    
        Init_RxMes();
        // 为接收下一包数据做好准备
        HAL_CAN_Receive_IT(&Can_Handle, CAN_FIFO0);
    }
    ```

  - 轮询方式接收

    在需要接收数据的地方直接调用HAL_CAN_Receive即可，在接收完成的时候会返回HAL_OK。

### 1.3、下载验证

将编译好的程序下载到开发板，运行程序，查看输出的log，如果能看到打印出接收的数据，则表示测试通过。

## 二、双机通讯

例程路径：ls_sdk\examples\peripheral\can\CAN_Normal

### 2.1、功能说明

本示例演示了如何使用两块开发板进行CAN通讯。需要将芯片的CAN_TX 和CAN_RX两个引脚与CAN收发器相连，收发器使用CANH及CANL引脚连接到CAN总线网络中。

### 2.2、软件配置

与回环模式下的配置基本一致，只有模式不同

```c
Can_Handle.Init.Mode = CAN_MODE_NORMAL;
```

### 2.3、下载验证

将编译好的程序下载到开发板，运行程序，查看输出的log，如果两块板子能看到打印出接收的数据，则表示测试通过。