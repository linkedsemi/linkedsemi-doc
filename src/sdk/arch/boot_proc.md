# 启动流程

系统启动分为三个阶段：

Primary Bootloader -> Secondary Bootloader （SBL）-> Application

## Primary Bootloader

此部分程序被固化在ROM中，芯片生产后就不可修改，也称之为Boot ROM。

系统启动首先执行这部分程序，简要流程如下：

1. 使能Boot PIN内部下拉电阻
2. 读取Boot PIN电平，根据电平选择启动模式

| Boot PIN Voltage | Mode            |
| ---------------- | --------------- |
| Low              | Boot From Flash |
| High             | Boot From UART  |

3. 从Flash读取必要配置信息（包括配置信息和SBL存储地址、大小）
3. 若为Flash启动模式，则根据SBL存储地址和大小，将其拷贝到RAM中，跳转到SBL代码开始执行；若为UART启动模式，则等待接收UART数据进行交互。

| IC     | Boot PIN | UART TX | UART RX |
| ------ | -------- | ------- | -------- |
| LE501X | PB14     | PB00    | PB01     |



## Secondary Bootloader

SBL存放在Flash中，被Boot ROM拷贝到RAM中执行。

SBL会执行下述逻辑：

- 初始化Flash控制器
- 加载芯片校准参数
- 检查OTA状态
  - 是否需要执行OTA固件拷贝
  - 是否需要跳转到单区OTA镜像执行

最后跳转到应用执行。

## Application 

- [BLE(MESH) Bare Metal App](./bare_metal_app)
- [BLE RTOS App](./rtos_app)
- MCU Bare Metal App