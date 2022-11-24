.. dac_ref:

DAC
==============

DAC是Digital-to-Analog Converter的缩写。指数/模转换器或者数字/模拟转换器。是指将数字信号转换成模拟信号。

一、 DAC的主要特征：
>>>>>>>>>>>>>>>>>>>>>>

#. 2个DAC转换器：每个转换器对应1个输出通道
#. 12位模式下的左或右数据对齐
#. 同步更新功能
#. 噪声波和三角波的产生
#. 双DAC通道独立或同时转换
#. 每个通道都有DMA功能
#. 外部触发转换
#. DAC输出连接到芯片上的外围设备

DAC系统框图如下：
>>>>>>>>>>>>>>>>>>>

二、 DAC接口介绍
----------------------
2.1  IO初始化
----------------------

    调用IO 的初始化接口，可以配置具体的IO的DAC模拟功能。

.. code :: 

    void pinmux_dac1_init(void);
    void pinmux_dac2_init(void);

.. note ::

    每一个DAC的通道都有其对应的IO，不可随意映射。具体的对应关系如下：

    ========= =============
      GPIO      ANA_FUNC1
    ========= =============
    GPIO_PA07    DAC_CH1
    GPIO_PC04    DAC_CH2
    ========= =============

2.2  DAC_HandleTypeDef 结构体初始化
------------------------------------

.. code ::

    typedef struct __DAC_HandleTypeDef
    {
        reg_dac_t   *Instance;        /*!< Register base address */
        uint8_t     DACx;
        uint8_t     DAC1_Trigger;         /*!< trigger selection */
        uint8_t     DAC1_wave;            /*!< DAC noise/triangle wave generation */ 
        uint32_t    DAC1_Mamp;            /*!< Set the amplitude of the triangular wave */
        uint8_t     DAC2_Trigger;         
        uint8_t     DAC2_wave;            
        uint32_t    DAC2_Mamp;           
        void        *DMAC_Instance; 
        union{
            struct DacDMAEnv DMA;
            struct DacInterruptEnv Interrupt;
        }Env;
    }DAC_HandleTypeDef;


**参数说明**

（1） DAC寄存器结构化处理(Instance)

    . 目前Gemini支持两个DAC

    . DAC1和DAC2的基地址相同：0x4008d400

（2） 选择要使用哪一个DAC(DACx)

（3） 选择触发器(DACx_Trigger)

    - 触发器选择说明：
  
    .. code ::

        #define   BASE_TIMER_TRGO       0x00000000U
        #define   GENERAL_TimerA_TRGO   0x00000001U
        #define   GENERAL_TimerB_TRGO   0x00000002U
        #define   GENERAL_TimerC_TRGO   0x00000003U
        #define   ADVANCE_Timer1_TRGO   0x00000004U
        #define   ADVANCE_Timer2_TRGO   0x00000005U
        #define   PIP_OUTPUT_CHANNEL    0x00000006U
        #define   SOFTWARE_TRIG         0x00000007U

（4） 选择是输出电压，还是产生波形(DACx_wave)

    - 可选参数如下：

    .. code ::

      #define     No_Wave                 0x00000000U             /* wave generation disabled */
      #define     Noise_Wave              0x00000001U             /* Noise wave generation enabled */
      #define     Triangle_Wave           0x00000002U             /* Triangle wave generation enabled */

（5） 选择波形的峰值(DACx_Mamp)

    - 峰值选择说明：

    .. code ::

      #define   triangle_amplitude_1                  0x00000000U
      #define   triangle_amplitude_3                  0x00000001U
      #define   triangle_amplitude_7                  0x00000002U
      #define   triangle_amplitude_15                 0x00000003U
      #define   triangle_amplitude_31                 0x00000004U
      #define   triangle_amplitude_63                 0x00000005U
      #define   triangle_amplitude_127                0x00000006U
      #define   triangle_amplitude_255                0x00000007U
      #define   triangle_amplitude_511                0x00000008U
      #define   triangle_amplitude_1023               0x00000009U
      #define   triangle_amplitude_2047               0x0000000aU
      #define   triangle_amplitude_4095               0x0000000bU 

（6） DAC转换的触发方式(Env)

2.3  DAC模块初始化
----------------------

.. code ::

   HAL_DAC_Init(DAC_HandleTypeDef *hdac);

2.4  将数字量写入已选择的数据对齐格式的寄存器
-----------------------------------------------

.. code ::

    HAL_DAC_SetValue(DAC_HandleTypeDef *hdac, uint32_t Alignment, uint32_t Data);

.. note ::

    #. 此函数在DMA模式和输出三角波、噪声波的条件下不可以。
    #. Alignment:选择数据的对齐格式，具体如下：
        
    .. code ::

      #define DAC1_ALIGN_12B_R                    0x00000000U
      #define DAC1_ALIGN_12B_L                    0x00000004U
      #define DAC1_ALIGN_8B_R                     0x00000008U

      #define DAC2_ALIGN_12B_R                    0x00000000U
      #define DAC2_ALIGN_12B_L                    0x00000004U
      #define DAC2_ALIGN_8B_R                     0x00000008U

      #define DAC12_ALIGN_12B_RD                  0x00000000U
      #define DAC12_ALIGN_12B_LD                  0x00000004U
      #define DAC12_ALIGN_8B_RD                   0x00000008U

2.5  获取数据输出寄存器中的值
--------------------------------


    .. code ::

        HAL_DAC_GetValue(DAC_HandleTypeDef *hdac);
    
    .. note ::

        使用DAC1时返回dac_dor1寄存器中的数据；

        使用DAC2时返回dac_dor2寄存器中的数据；

        同时使用两个DAC时，此函数返回值为0。

2.6  数据转换——DMA模式
--------------------------------

    .. code ::
     
        HAL_StatusTypeDef HAL_DAC_Start_DMA(DAC_HandleTypeDef* hdac, uint32_t Alignment, uint32_t* pData, uint32_t Length,void (*Callback)(DAC_HandleTypeDef* hdac));

    .. note ::

        Alignment : 数据对齐格式。

        pData     : 由内存搬运到外设的的数字量。

        length    : pData的大小，使用sizeof计算。
        
        Callback  : 由用户实现。

2.7  反初始化
--------------

反初始化DAC模块
...............

通过反初始化接口,应用程序可以关闭DAC模块。
