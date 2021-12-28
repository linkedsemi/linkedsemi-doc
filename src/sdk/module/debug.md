# Debug

## Assertion

SDK中有如下三类断言：

- ls_assert

  SDK源码中的断言，意味着产生了”不可能“出现的条件。会输出断言产生的具体源文件代码位置。

- ls_ram_assert

  SDK源码中的断言，但发生在RAM中执行的函数内。由于RAM中的函数常常不可访问Flash指令，因此专门区分出RAM函数内发生的断言。

- stack_assert_c

  协议栈二进制固件中的断言。会抛出断言级别、断言发生的返回地址和相关参数。需要原厂分析具体原因。只有**LVL_ERROR**级别的断言会停止程序。

*当量产产品经过**充分测试验证**后，为了避免极低概率的系统死机，可以将上述断言中程序停止死循环的代码，改为系统复位。但**务必不要**在量产产品充分测试验证前这样处理，因为会造成异常现场丢失，不利于排查问题。*

## LOG

特点

- 支持日志分级全局和源文件静态编译控制
- 支持定义源文件日志标签
- 支持多种输出，包括：J-LINK RTT、UART

```c
#define LVL_ERROR 1
#define LVL_WARN 2
#define LVL_INFO 3
#define LVL_DBG  4
```

系统定义了上述四个日志级别，分别对应四个输出宏：

```c
LOG_E(...)
LOG_W(...)
LOG_I(...)
LOG_D(...)
```

使用格式等同于`printf`。

### GLOBAL_OUTPUT_LVL

对于全局日志输出级别控制，小于等于`GLOBAL_OUTPUT_LVL`级别的日志会被编译。

默认情况下，

```c
#ifndef GLOBAL_OUTPUT_LVL
#define GLOBAL_OUTPUT_LVL     LVL_DBG
#endif
```

### 文件级标签和输出级别控制

源文件使用`LOG_x`宏，需要包含log.h头文件

```c
#define LOG_TAG ...
#define LOG_LVL	...
#include "log.h"
```

在包含log.h头文件前，可以定义`LOG_TAG`和`LOG_LVL`。

`LOG_TAG`为当前文件的标签，内容为字符串，当前文件使用`LOG_x`宏输出日志，输出内容中均会包含该标签。若未定义，默认值为`"NO_TAG"。`

`LOG_LVL`为当前文件日志级别。当前文件中所有小于等于`LOG_LVL`级别的日志会被编译。默认值为`LVL_DBG`。

### 裸输出

- `LOG_RAW`

  格式等同于`printf`。输出内容不含任何参数之外的信息。编译件：`GLOBAL_OUTPUT_LVL!=0`

- `LOG_HEX(data_pointer,data_length)`

  输出data_pointer地址开始data_length长度的数据的十六进制格式。低地址在前。编译条件：`GLOBAL_OUTPUT_LVL!=0`

### Multiple Backends

```c
#define JLINK_RTT          1
#define UART_LOG           2
#define LOG_BACKEND (JLINK_RTT|UART_LOG)
```

上述`LOG_BACKEND`定义在log.c文件中，即向JLINK_RTT和UART_LOG两种后端输出日志。

```c
#define LOG_UART_TXD (PB00)
#define LOG_UART_RXD (PB01)
```

定义了UART_LOG具体输出管脚。
