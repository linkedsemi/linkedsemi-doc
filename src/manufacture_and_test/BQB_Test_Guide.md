# BQB_Test_Guide



## 1. 1    BQB测试的必要性

BQB即Bluetooth Qualification Body（蓝牙认证机构），产品若具备[蓝牙功能](https://baike.baidu.com/item/蓝牙功能/98719)并且需在产品外观上标明蓝牙标志，则须通过蓝牙认证测试即BQB认证测试。

 

## 1. 2    开发板连接&烧录

**Pin脚连接描述：**

注：若开发板pin脚未标注SWDIO,SWCLK，默认参考配置如下【图1.2-1】

图1.2-1

![](./Picture/image-20220120000212426.png)

图1.2-2

**开发板与****Jlink****连接示意图：**

![](./Picture/image-20220120000616000.png)

图1.2-3

**开发板与****UART****连接示意图：**

![](./Picture/image-20220120000736505.png)

 

**测试固件获取：**

（1）通过网址下载：https://gitee.com/linkedsemi/ls_ble_sdk?_from=gitee_search

Hex文件路径：SDK根目录\build\le501x\examples

![](./Picture/contents.png)

（2）联系技术支持工程师获取烧录固件

 

**测试固件烧录：**

（1）通过JFlash烧录

使用教程请参阅：

https://ls-ble-sdk.readthedocs.io/zh/latest/getting_started/keil_debug.html#jflash

（2）通过UART（串口）方式烧录

使用教程请参阅：

https://ls-ble-sdk.readthedocs.io/zh/latest/Download_Tool_Usage/download_tool_introduce.html

 

 

## 1. 3    CMW500按键说明

![](./Picture/overview.png)

**①**![](./Picture/6.png)   ：复位键

**②** ![](./Picture/7.png)  ：打开测试选择界面

**③** ![](./Picture/8.png)  ：打开信号源选择界面

**④**![](./Picture/9.png)   ：打开&关闭信号源

**⑤** ![](./Picture/10.png)  ：数据微调（左旋、右旋）& 确认（旋钮中心按下)

**⑥**![](./Picture/11.png)   ：测试任务栏显示

**⑦** 侧边功能选键 ：用于选择此按键正对屏幕区域内的子功能选项




## 1. 4    建立连接

使用前请单击CMW500左上角 ![](./Picture/6.png)  按键，弹出对话框【图1.2-4】 

依次选择①②项，完成对CMW500的最佳预设值恢复。

图1.2-4

 ![](./Picture/12.png)

然后点击右侧面板 **SIGNAL GEN** 按键，勾选Bluetooth的子项**Signaling**【**旋钮右旋调整选择位置，按下旋钮中心为选择**】

加载后，即在底部任务栏区域出现Bluetooth Signaling标识【图1.2-5】***（若未出现，请单击靠近屏幕右下区域的 TASKS按键）***，接着单击屏幕底部【**侧边功能选键**】即进入Bluetooth Signaling设置页面。【图1.2-6】

 

图1.2-5

![](./Picture/13.png)

 

图1.2-6

 ![](./Picture/14.png)

a）右侧Burst Type选择为Low Energy，根据测试需求设置PHY。

b）左侧HW Interface选择为USB to RS232 adapter，Baud Rate设置为115200（实际以固件UART波特率设值为准）

c）选择Routing【**侧边功能选键**】，将会出现如  图1.2-7 所示的底部选择栏

(1)选择Routing(Output) 【**侧边功能选键**】设置RF Signal的输出端口（请选择带COM标识的端口，OUT端口仅支持CMW500端RF信号输出）

(2)选择External Att(Output) 【**侧边功能选键**】，设置当前RF线缆的线损。

请根据上述操作逻辑继续配置输入信号端口、线损。

图1.2-7

![](./Picture/15.png)

d）如下截图中 Virtual COM Port未识别到，请检查UART线缆连接状态。若连接正常还是未识别，需确认CMW500端【设备管理器】，确认该UART线缆的驱动是否正确安装。【图1.2-8】

图1.2-8

 ![](./Picture/16.png)

 

完成以上设置后，可以开始进行CMW500与开发板建立通信连接。

按仪器右上侧面板![](./Picture/17.png)键，设置的RF信号端口指示灯亮，显示界面底部出现Connection Check选项。（图1.2-9)

图1.2-9

![](./Picture/18.png)

选择Bluetooth Signaling 【**侧边功能选键**】，再选择Connection Check【**侧边功能选键**】，连接成功后Event Log将提示Start TX Test，约3秒后出现连接成功提示框。（如图1.2-10）

**注：初次连接，Loading Stack可能需要5s~6s**

图1.2-10

![](./Picture/19.png)

 

## 1. 5    TX配置&测试

选择仪器右侧面板按键 **MEASURE**  ，选择①，再选择底部的 **Bluetooth Multi Eval**【**侧边功能选键**】即可进入TX测试界面

图1.2-11

![](./Picture/20.png)

 

 

### 1. 5. 1     测试信号类型设置

在Multi Evaluation界面，依次选择①②③，选择蓝牙信号类型为Low Energy【即低功耗蓝牙】

图1.2-12

![](./Picture/21.png)

 

### 1. 5. 2     信号路径、线损、信道设置

（1）在Multi Evaluation界面，依次选择①②，设置Scenario【即：方案】为Combined Signal Path【即：组合信号路径场景】，设置完成后，请点击右侧面板的  **CLOSE** 键或鼠标点击小窗口右上角的 **X** 关闭当前设置小窗口。

图1.2-13

![](./Picture/22.png)

 

（2）选择RF Settings【**侧边功能选键**】底部选项栏将出现设置项

RF Routing：按①②③④操作顺序，设置信号输入输出端口【图1.2-14】

图1.2-14

![](./Picture/23.png)

 

External Attenuation：按①②③④操作顺序，设置射频线缆的输入输出线损【图1.2-15】

图1.2-15

![](./Picture/24.png)

 

Frequency/Channel：按①②③操作顺序，设置测试信号的信道(频率值根据信道自动更新) 【图1.2-16】

 

图1.2-16

![](./Picture/25.png)

 

 

 

（3）选择Input Signal【**侧边功能选键**】底部选项栏将出现设置项【图1.2-17】

Burst Type：测试信号类型

Pattern Type：测试包的模式类型

PayLoad Length：测试包的长度

图1.2-17

![](./Picture/26.png)

### 1. 5. 3     测试结果&视图设置

完成上述设置后，按仪器右侧面板  ![](./Picture/17.png) 键，即可开始测试。

（1）Pass示例

图1.2-18

![](./Picture/27.png)

（2）Fail示例

图1.2-19

![](./Picture/28.png)

（3）测试中可能变更测试结果展示视图，将更侧重显示选择的测试数据

按下图①②③顺序选择，即可设置Display

Overview 整体结果概览，会将以下几种视图叠放展示

Power vs Time  发射功率相关的数据

Frequency Deviation  信号频率偏差相关的数据

Modulation Scalars  信号调制相关的标量数据

图1.2-20

![](./Picture/29.png)

 

## 1. 6    RX配置&测试

### 1. 6. 1     RX测试环境

（1）RX测试若要获得较为准确的结果需在微波暗室或屏蔽箱中进行，以确保外界杂波信号不会对测试造成干扰。

（2）基于我司使用的屏蔽箱，内部（左侧）&整体连接（右侧）示意图如下：

图1.2-21

![](./Picture/30.png)

 

选择仪器右侧面板按 **MEASUER**  键，勾选①，再选择底部的 Bluetooth RX Meas【**侧边功能选键**】

图1.2-22

![](./Picture/31.png)

即可进入RX测试界面

图1.2-23

![](./Picture/32.png)

上图页面可完成以下设置：

①测试信号类型设置

②测试速率设置

③测试信道设置

④CMW发射信号电平值设置

⑤测试包长度设置

⑥测试包类型设置

⑦测试包数量设置

### 1. 6. 2     信号路径、线损、信道设置

选择RF Settings【**侧边功能选键**】即可显示射频常规设置项（输入输出端口、线损、信道频率等）【图1.2-24】

图1.2-24

![](./Picture/33.png)

### 1. 6. 3 单次PER测试

完成上述设置后，按仪器右侧面板 ![](./Picture/17.png)  键，即可开始测试。

PER测试分为2个部分，**PER**和**PER Search**

PER Search说明请在 **1.6.4 PER Search测试**中查看



Pass示例

图1.2-25

![](./Picture/34.jpeg)

截图中：

①代表测试状态，RUN表示进行中  RDY代表测试完成，仪器处于测试预备状态

②处值为PER（Packet Error Rate）即收到的错包、漏包数占总包数的百分比（PER低于30.8%为Pass）

③处值为收到的正确包数

Fail示例

图1.2-26

![](./Picture/35.jpeg)

 

### 1. 6. 4 PER Search测试

PER测试分为2个部分，**PER**和**PER Search**

PER Search 用于从设定的TX Level值开始测试RX指标，若单次测试的PER＜30.8%。则逐步减小TX Level值**（每次减小的步进值可在Level Step项设置）**，继续进行RX指标测试，直到RX的PER(误包率)≥**30.8%**则停止。

单次PER 测试说明请在 **1.6.3 单次 PER测试**中查看



 

**设置项：**

Start Level：CMW发射的信号电平值

Level Step：每完成一次Search后，若PER仍＜30.8%，Start Level变化的步进值。步进值越小，测量得到RX灵敏度精度则相对越精确。

 

截图测试结果说明：

表明该模组在Start Level为 -97 dBm，Search Result为-97.5dBm时,，PER开始 ≥30.8%

（即芯片模组的RX灵敏度约为-97.5dBm）

图1.2-27

 ![](./Picture/36.jpeg)
