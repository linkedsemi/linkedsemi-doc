# i2c_it 使用示例

例程路径: ls_sdk\examples\peripheral\i2c\i2c_it

本示例主要用于演示如何使用i2c it相关接口进行数据收发。

## 一、程序基本配置：

### 1.1 宏定义说明

宏定义包括：

```c
/*i2c角色定义*/
#define I2C_MASTER_ROLE 0
#define I2C_SLAVE_ROLE 1
#define CURRENT_ROLE I2C_SLAVE_ROLE
/*i2c速率定义*/
#define I2C_SPEED I2C_SPEED_FAST_400K
/*i2c地址定义*/
#define I2C_ADDRESS 0x50
/*i2c传输buffer大小定义*/
#define BUFFER_LEN 255
/*i2c demo运行时是否使能模拟更高优先级中断，通常配置为0*/
#define SIMU_IRQ_EN 0
```

### 1.2 其他配置

该例程使用到的全局变量包括：

```c
/*数据收发状态标志位*/
static volatile uint8_t Com_Sta = start;
/*i2c用户数据*/
static I2C_HandleTypeDef I2cHandle;
/*数据接收buffer*/
static uint8_t aRxBuffer[BUFFER_LEN];
/*数据发送buffer*/
static uint8_t aTxBuffer[BUFFER_LEN];
/*总线IO配置，PB08配置为SCL，PB09配置为SDA*/
pinmux_iic1_init(PB09, PB08);
```

## 二、程序运行流程说明

### 2.1 初始化

```c
/*系统初始化*/
sys_init_none();
/*调试toggle IO初始化，非必须*/
toggle_debug_IO_init();
/*pinshare配置，PB08为SDA，PB09为SCL*/
pinmux_iic1_init(PB09, PB08); 
/*应用IIC参数配置初始化*/
iic_init();
/*IIC初始化*/
if (HAL_I2C_Init(&I2cHandle) != HAL_OK)
{
    Error_Handler();
}
```

### 2.2 运行流程

IIC的测试为无限循环的主从机数据交互测试，具体测试步骤为：

1. 设置测试数据长度test_len初始值为1字节
2. 主机/从机生成test_len长度的随机数，存在aTxBuffer里（主机有效，从机生成的数据无实际作用），aRxBuffer里内容初始化为0xff
3. 如果test_len在(100, 200)之间，主机端会生成一个在(0, 255-test_len)之间的随机数，之后主机进入发送流程，将aTxBuffer里长度为test_len+extra_data_len的数据write到从机，从机接收并存在aRxBuffer里。而从机只会接收test_len长度，因此在extra_data_len==0时，主机端会调用HAL_I2C_MasterTxCpltCallback，而extra_data_len>0时，会调用HAL_I2C_ErrorCallback，且满足extra_data_len == I2cHandle.XferCount条件。相应的从机会调用HAL_I2C_SlaveRxCpltCallback
4. 发送完成后，主机再进入接收流程，接收从机发送过来的数据，长度为test_len。相应的，从机进入发送流程，将之前收到的test_len长度数据回发给主机。完成后，主机调用HAL_I2C_MasterRxCpltCallback，从机调用HAL_I2C_SlaveTxCpltCallback
5. 主机端将收到的和发送的数据进行比较，所有数据内容一致则pass，进入下一轮，否则判定为fail
6. 若进入下一轮，主、从机端会根据收到的数据里第一个字节(aRxBuffer[0])，作为下一轮数据收发的长度，赋值给test_len，这样既能保证主从双方下一轮测试都有一个共同的长度值，同时能做到长度随机。如果test_len为0，则改为1
7. 生成test_len后，返回步骤2，进行下一轮测试
8. 测试的过程中，测试板上相应的IO会拉高拉低，对应的LED会持续闪烁。如果测试中fail，LED会停止闪烁

## 三、操作步骤及结果：

### 3.1 操作步骤

1. 运行demo前，需先准备好硬件环境，包括主从两个demo板，根据demo配置的IO对接，以及通过VDD3.3上拉，上拉电阻建议配置为1KΩ
2. 修改CURRENT_ROLE为从机，配置好其他参数，将编译生成的hex下载到从机demo板，Reset从机，使其提前运行
3. 修改CURRENT_ROLE为主机，配置好其他参数（需要与从机一致），将编译生成的hex下载到主机demo板，Reset主机

***注意：测试过程中需保证环境干净，不可以有毛刺，否则可能会造成运行失败***

### 3.2 测试结果

主从机数据收发反复进行，无限循环，主从机LED灯会持续闪烁

## 四、其他

- 驱动鲁棒性验证选项：demo里创建了一个硬件定时器，且将其中断优先级设置为高于i2c。在该demo运行过程中，timer会产生updated中断，并调用HAL_TIM_PeriodElapsedCallback函数，在该函数里，根据生成的随机数值，CPU会delay 1-1000us不等，同时会修改下一次timer updated产生的时间点，该时间在3000-10000us之间。此选项默认关闭，可以通过将SIMU_IRQ_EN设置为1来打开。此选项打开与否不会影响demo的运行结果
- i2c主从数据收发，应用要做到主从机单次收发长度一致，否则可能出现总线被拉死的场景。而一旦出现主从机配置的数据收发长度不一致，则会出现以下两种情况：
  1. 主机write数据，从机负责控制每个字节后的ACK/NACK。如果主机配置的数据发送长度大于从机配置的数据接收长度时，从机会在本地数据接收的最后一个byte后发送NACK到主机，这种情况下主机即便还有数据没有发完，也必须发出stop来结束当前传输；如果主机配置的数据发送长度小于从机配置的数据接收长度，尽管主机会在发送完本地最后一个byte后接收到ACK，但由于所有数据已经传送到从机，因此主机会发出stop来结束当前传输
  2. 主机read数据，主机负责控制每个字节后的ACK/NACK。如果主机配置的数据接收长度大于从机配置的数据发送长度时，从机会在本地数据发送的最后一个byte收到ACK，此时由于从机数据已经发送完成，又无法发送NACK通知主机，因此会出现***从机驱动无法处理的场景，总线会被拉死，此种场景禁止出现***；如果主机配置的数据接收长度小于从机配置的数据发送长度时，主机会在本地数据接收完成后发送NACK以及stop来结束当前传输

