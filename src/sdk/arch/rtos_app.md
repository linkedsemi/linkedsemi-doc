# RTOS应用

RTOS系统配置为抢占式。

BLE RTOS应用最小任务集如下：

|            | Priority    | Description                                       |
| ---------- | ----------- | ------------------------------------------------- |
| TIMER TASK | Highest     | Handles all RTOS timers and its' related messages |
| BLE TASK   | Highest - 1 | Handles all BLE protocols and events              |
| IDLE TASK  | Lowest      | Handles the low power flow                        |

所有BLE事件的产生都在BLE TASK上下文。

由于BLE API不是线程安全的，因此除非确定不会重入的情况下，所有BLE API都需要在BLE TASK上下文调用。

## OS Tick & Tickless Idle

OS Tick利用BLE硬件定时器产生，节省了其他硬件定时器资源，同时基于BLE硬件定时器的补偿机制，可以实现准确的Tickless Idle后Tick补偿。

BLE广播、连接应用在固定周期的数据收发后，系统进入IDLE TASK，若LP0休眠检查通过，则进入LP0，此时OS Tick不会再周期性地更新。系统唤醒后，会重新使能周期性OS Tick，并根据OS Tick实际暂停时间，补偿OS Tick计数，以保证OS的计时机制准确。