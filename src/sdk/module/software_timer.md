# 软件定时器

SDK提供两类软件定时器：

- builtin_timer
- sw_timer

两者可用性区分如下：

|        | BLE App       | MCU App  |
| ------ | ------------- | -------- |
| LE501X | builtin_timer | sw_timer |

## BUILTIN_TIMER

builtin_timer基于LE501X BLE协议栈内部定时器封装而成，因此只能在LE501X BLE应用环境下使用。

builtin_timer支持创建任意数量的定时器，具体数量受制于应用分配的协议栈堆大小。

当定时器超时后，相应的硬件中断会抛出超时消息，超时消息在协议栈大循环中被处理，即定时器超时回调在协议栈**大循环中调用**。因此当系统负载较重，即大循环消息队列中有大量待处理消息时，builtin_timer的回调会被延迟。

builtin_timer可以在LP0下正常工作，即builtin_timer存在并且运行的情况下，系统能够进入LP0，且能够在builtin_timer超时前唤醒，超时后调用回调。

## SW_TIMER

sw_timer是一个充分考虑到可移植性的软件定时器，其实现分为前端、后端。前端源文件即sw_timer.c，管理定时器队列的更新，实现定时器的启动，停止，配置，查询。后端源文件通常为sw_timer_port.c，负责sw_timer在具体硬件定时器源上的适配，以及硬件定时器源在LP0下的休眠和唤醒补偿。

LE501X MCU应用场景下，sw_timer基于BLE硬件基带定时器实现。

sw_timer基于静态内存实现。开发者可以定义不限数量的sw_timer实例结构体，通过将结构体指针作为API参数，来操作所定义的sw_timer。

sw_timer超时回调在**中断里调用**。

sw_timer可以在LP0下正常工作，即sw_timer存在并且运行的情况下，系统能够进入LP0，且能够在sw_timer超时前唤醒，超时后调用回调。
